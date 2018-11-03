# Editing user data

Now let's try to edit user content. Remember, we are going to write a voting system.


## `data.json` files

Now let's write a function which will add a question.


First of all, we will take our `js/files.js` program and include it to `index.html`:

    function readFile(file, callback) {
        zeroFrame.cmd("fileGet", [file, false], callback);
    }
    
    function writeFile(file, content, callback) {
        zeroFrame.cmd("fileWrite", [file, base64Encode(content)], callback);
    }
    
    function base64Encode(content) {
        content = encodeURIComponent(content); // Split to bytes in % notation
        content = unescape(content); // Join % notation as bytes (not as chars)
        return btoa(content);
    }


Now, let's write `addQuestion()` function in `js/votes.js`:

    function addQuestion(question, answers, callback) {
        ...
    }


If somebody wants to add a question, he has to be authorizated with ZeroID:

    function addQuestion(question, answers, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }
            
            ...
        });
    }


Let's open JSON file of user:

    function addQuestion(question, answers, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }
            
            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";
                
                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }
                
                ...
            });
        });
    }

Notice that we try to open file with user address. User address is its public key for ZeroID. If user uses KaffieID, public key and username (`user` property) will be different.


Now we add question, save JSON and publish user's `content.json`:

    function addQuestion(question, answers, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }
            
            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";
                
                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }
                
                var id = content.next_question_id;
                content.questions.push({
                    id: id++,
                    question: question,
                    answers: answers.join("\n"),
                    date_added: Math.floor(Date.now() / 1000)
                });
                
                content = JSON.stringify(content);
                
                writeFile("data/users/" + user.address + "/data.json", content, function() {
                    zeroFrame.cmd("sitePublish", {
                        inner_path: "data/users/" + user.address + "/content.json"
                    }, function() {
                        callback(id);
                    });
                });
            });
        });
    }

Notice that we don't pass `privatekey` to `sitePublish` command. That's because `privatekey: null` means "sign using user's private key".

We also sign `data/users/{address}/content.json`, but it is possible that it does not exist (eg. user added question first time). Hopefully, ZeroNet creates a `content.json` for us if it does not exist.


Let's check our code. Open DevTools, reload page and type the following in the console:

    addQuestion("What's nofish's name?", ["nofish", "Tomas", "Jack"], console.log.bind(console));

Authorizate using ZeroID (if you are asked) and watch `data/votes.db`.


## Answers

Try to write `addAnswer()` function for yourself first.

Answer:

    function addQuestion(questionId, answerId, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }
            
            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";
                
                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }
                
                content.answers[questionId] = answerId;
                
                content = JSON.stringify(content);
                
                writeFile("data/users/" + user.address + "/data.json", content, function() {
                    zeroFrame.cmd("sitePublish", {
                        inner_path: "data/users/" + user.address + "/content.json"
                    }, callback);
                });
            });
        });
    }


Note that we have similar functions. Let's write a library function instead:

    function editUserData(handler, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }
            
            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";
                
                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }
                
                handler(content);
                
                content = JSON.stringify(content);
                
                writeFile("data/users/" + user.address + "/data.json", content, function() {
                    zeroFrame.cmd("sitePublish", {
                        inner_path: "data/users/" + user.address + "/content.json"
                    }, callback);
                });
            });
        });
    }
    
    function addQuestion(question, answers, callback) {
        var id;
        editUserData(function(content) {
            id = content.next_question_id;
            
            content.questions.push({
                id: content.next_question_id++,
                question: question,
                answers: answers.join("\n"),
                date_added: Math.floor(Date.now() / 1000)
            });
        }, function() {
            callback(id);
        });
    }
    
    
    function addAnswer(questionId, answerId, callback) {
        editUserData(function(content) {
            content.answers[questionId] = answerId;
        }, callback);
    }


## Reading data

Now we will write `getQuestionList()`, `getQuestion()` and `getAnswers()` functions:

    function getQuestionList(sort, callback) {
        if(sort == "popular") {
            zeroFrame.cmd("dbQuery", ["SELECT questions.*, CASE WHEN answers.answer_count IS NULL THEN 0 ELSE answers.answer_count END AS answer_count FROM questions LEFT JOIN (SELECT question_id, COUNT(*) as answer_count FROM answers GROUP BY question_id) AS answers ON (answers.question_id = questions.id) ORDER BY answers.answer_count DESC, questions.date_added DESC LIMIT 0, 10"], callback);
        } else if(sort == "latest") {
            zeroFrame.cmd("dbQuery", ["SELECT * FROM questions ORDER BY date_added DESC LIMIT 0, 10"], callback);
        }
    }
    
    function getQuestion(id, callback) {
        zeroFrame.cmd("dbQuery", ["SELECT * FROM questions WHERE id = " + id], function(questions) {
            zeroFrame.cmd("siteInfo", [], function(siteInfo) {
                if(siteInfo.cert_user_id) { // User logged in
                    zeroFrame.cmd("dbQuery", ["SELECT answers.*, json.* FROM answers, json WHERE json.directory = \"users/" + siteInfo.auth_address + "\" AND answers.json_id = json.json_id AND answers.question_id = " + id], function(answer) {
                        if(answer.length) {
                            questions[0].answered = answer[0].answer_id;
                            
                            getAnswers(id, function(answers) {
                                questions[0].answers = answers;
                                callback(questions[0]);
                            });
                        } else {
                            questions[0].answered = -1;
                            callback(questions[0]);
                        }
                    });
                } else {
                    questions[0].answered = -1;
                    callback(questions[0]);
                }
            });
        });
    }
    
    function getAnswers(id, callback) {
        zeroFrame.cmd("dbQuery", ["SELECT answer_id, COUNT(*) as answer_count FROM answers WHERE question_id = " + id + " GROUP BY answer_id"], function(answers) {
            var result = {};
            for(var i = 0; i < answers.length; i++) {
                result[answers[i].answer_id] = answers[i].answer_count;
            }
            callback(result);
        });
    }

Now, core is finished. Only layout is left. As usual, [here](downloads/voting.html) is an example.
