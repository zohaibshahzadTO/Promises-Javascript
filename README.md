# Promises (Javascript)

I typically have always programmed using callbacks but for a while there has also been something called 'promises' which a few years ago you could import to your javascript program. Now as of ES2015, promises are native to how JS works.

# What are they and how do they work?

A promise is an object. So lets say we have a **loadJSON** function which expects a url and a callback function. Another reason we can get stuck in callback hell is because we might need multiple callback functions for multple things like error callbacks, etc. Code would become messy. 

        loadJSON(url, callback);

Instead we can have **let promise = loadJSON(url)**. The idea is that instead of passing a function a callback, we ask a function for a promise. Instead we'll use a function called "fetch" which is native to javascript in the browser that supports promises. This gives us a promise and once we have that, we have it as an object. Its an object that can be in a certain state: pending (waiting to get data back from API), fulfilled (successfully retrieved results from API), or rejected (some error has happened). If we have that object (promise), we say, "hey promise, you promised me something, im waiting, whats your status?"

If we do this, we actually dont need to query the promise continuously. We can use something called: **then()** or **catch()**. 

**then():** is a function that receives a function to be executed (kind of like a callback), when it has been fulfilled. 

**catch():** receives a function to be executed if and when it has been rejected.

Lets see where the function - fetch is coming from. Fetch is just a function that you give a url and it fetches content from that url and it gives a promise back.

        let wordnikAPI = "..."
        let giphyAPI = "...."

        function setup() {
            noCanvas();

            let promise = fetch(wordNikAPI);
            promise.then(gotData);
            promise.catch(gotErr);

            function gotData(data) {
                console.log(data);
            }

            function gotErr(err) {
                console.log(err);
            }
            console.log(promise); 
        }
	
Fetch function has weird properties. We got results back but actually the results need to be converted into a format we can use.

If we mess with the url, we'll actually get an error. This is the idea, we get a promise and give it a fucntion with **then()** for when its successful and then give a function for when there's an error under **catch()**. 

Lets condense this down with anonymous functions. Anonymous functions are functions that are dynamically declared at runtime. They're called anonymous functions because they aren't given a name in the same way as normal functions. Anonymous functions are declared using the function operator instead of the function declaration.

        fetch(wordnikAPI) 
            .then(function(data) {
                console.log(data);
            }).catch(function(err) {
                console.log(err);
            });

Lets use the arrow syntax:

        fetch(wordnikAPI) 
            .then(data  => console.log(data))
            .catch(err => console.log(err));


Like we mentioned earlier. Fetch function has weird properties. We got results back but actually the results need to be converted into a format we can use. We can do that by sing the **.json()** function.

The thing with the **.json()** function is that it also returns a promise.  


        fetch(wordnikAPI) 
            .then(response  => response.json())
            .then(json => console.log(json))
            .catch(err => console.log(err));

With the above, we no longer have callback hell, we simply have a sequences of lines that should be performed in order to get our desired result (retrieve data -> convert to json, catch any errors in the process).


Now that we have a random word being retrieved everytime the page is refreshed, we want to also show a giphy associated for that particular word. 

ES5 Version:

        fetch(wordnikAPI) 
            .then(response  => response.json())
            .then(json => {
                createP(json.word);
                fetch(giphyAPI + json.word);	
            })
            .catch(err => console.log(err));

So now we want to fetch the word 'wordnikAPI' and take the response and convert it into json, once I have a json I want to create the paragraph and then fetch from the giphyAPI. What do I want to do now? We'll then take THAT response and convert it into json, then take the json and create the image. 

        function setup() {
            noCanvas();

            fetch(wordnikAPI) 
                .then(response  => response.json())
                .then(json => {
                    createP(json.word);
                    fetch(giphyAPI + json.word);	
                })
                .then(response => response.json())
                .then(json => {
                    createImg(json.data[0].images['fixed_height_small'].url)
                })
                .catch(err => console.log(err));
        }


If you run that snippet of code, we'll actually get an error. The second **.then(response => response.json())** in that snippet was reported to have an error.

Promises are chainable. What are the things that return a promise? Fetch() returns one, **.json()** also returns a promise. So how come if the first **.json()** returns a promise and the subsequent code work but the second time we use that line it fails? 

Here's the thing, in this subsequent chain of promises, we need to explicitly say **return** in order chain all of the events. 

        function setup() {
            noCanvas();

            fetch(wordnikAPI) 
                .then(response  => response.json())
                .then(json => {
                    createP(json.word);
                    return fetch(giphyAPI + json.word);	
                })
                .then(response => response.json())
                .then(json => {
                    createImg(json.data[0].images['fixed_height_small'].url)
                })
                .catch(err => console.log(err));
        }

If we have to return a promise in order to have the next **then()** sequence properly. The beauty of all of this is that the **.catch()** method will catch errors for any of the **then()** functions and this is great because when using callback functions you have to embed a seperate error callbacks for every single thing you do for a sequence like that. 