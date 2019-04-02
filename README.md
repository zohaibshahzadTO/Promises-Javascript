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


# Making our Promise

        function setup() {
            noCanvas();
            setTimeout(sayHello, 1000);
        }

        function sayHello() {
            createP('hello');
        }

Right now we have a function thats asynchronous in which we pass a callback. What if wanted to create our version of the *setTimeout()* method that returned a promie instead?

We're now going to create a function called delay:

        function setup() {
            noCanvas();
            delay(1000);
        }

        function delay(time) {
            setTimeout(sayHello, time);
        }

        function sayHello() {
            createP('hello');
        }

Of course we wont be doing in this a practical setting because right now we're just embedding the setTimeout method in a function called delay and then calling that delay function in another. But now this is where we start out promise.

        function setup() {
            noCanvas();
            delay(1000)
                .then(() => createP('hello')
                .catch((err) => console.log(err));
        }

        function delay(time) {
            setTimeout(sayHello, time);
        }

        function sayHello() {
            createP('hello');
        }


This will give us an error because theres no promise thats been returned. So my delay function needs to return a new promise. 

How do we make a promise. We need to pass pathways for the resolution and rejection for the promise.

        function setup() {
            noCanvas();
            delay(1000)
                .then(() => createP('hello')
                .catch((err) => console.log(err));
        }

        function delay(time) {

            function dealWithPromise(resolve, reject) {

            }

            return new Promise(dealWithPromise);
            // setTimeout(sayHello, time);
        }

        function sayHello() {
            createP('hello');
        }


The *dealWithPromise() function* we just created is to solely handle resoluton and rejection of the promise and that function is returned within the new Promise. 

But typically it would be written as an anonymous function (see below).

        function setup() {
            noCanvas();
            delay(1000)
                .then(() => createP('hello')
                .catch((err) => console.log(err));
        }

        function delay(time) {

            return new Promise((resolve, reject) => {

            });
            // setTimeout(sayHello, time);
        }

        function sayHello() {
            createP('hello');
        }


Now this is what we're most likely to see. We want this delay function to return a new promise and we need to provide pathways on handling resolution and rejection.

        function setup() {
            noCanvas();
            delay(1000)
                .then(() => createP('hello')
                .catch((err) => console.log(err));
        }

        function delay(time) {
            return new Promise((resolve, reject) => {
                if (isNaN(time)) {
                    reject(new Error('delay requires a valid number');
                }
                setTimeout(resolve, time);
            });
        }


# ES8 - Async/Await

We'll begin with the functions we wrote above:

        function setup() {
            noCanvas();
            delayES8(1000)
                .then(() => createP('hello')
                .catch((err) => console.log(err));
        }

        function delayES8() {

            // This fn returns a promis
            await delay(time);
            
            return;
        }

        function delay(time) {
            return new Promise((resolve, reject) => {
                if (isNaN(time)) {
                    reject(new Error('delay requires a valid number');
                }
                setTimeout(resolve, time);
            });
        }


If a function returns a promise. What returns a promise? The delay function returns a promise. If thats the case, we can use the keyword *await*.

*await*: just wait for the promise to resolve.

We'll get an error saying that *await* is only valid in an async function. We cant just use that word in any function. Basically we have to write a function that returns a promise. Instead of writing *return new Promise*, the *await* word just says "hey just do that stuff behind the scenes for me". 


        function setup() {
            noCanvas();
            delayES8(1000)
                .then(() => createP('hello')
                .catch((err) => console.log(err));
        }

        async function delayES8() {

            // This fn returns a promis
            await delay(time);
            
            return;
        }

        function delay(time) {
            return new Promise((resolve, reject) => {
                if (isNaN(time)) {
                    reject(new Error('delay requires a valid number');
                }
                setTimeout(resolve, time);
            });
        }

If we modify that functon with a async keyword, calling it an async function. Its going to execute asynchronously and return a promise after however many calls to *await* that I want. 


# ES8 - Async/Await (Con't)

Using the first example we used where we worked with two API's and tried getting suitable image for the word projected we'll delve a bit more in how we can utilize the **async** and **await** keywords:

  
  
Now let's use the **await** and **async** keywords. If I tag the **await** keyword in from fetch(wordnikAPI), it pretty much handles the fetch and .then() and await the result. If im writing a fn that calls await it must be labelled async. 

Recall that **fetch** and **then** methods both return promises.

So this was an example of writing a function called **wordGIF()** which it asynchronously steps through all of these async calls one at a time using the await keyword so we can sequence them, so when its done, we have the data we want and we can send it back. 

# Promise All

Now what if I want to make multiple calls to wordGIF() and I want them to retain something about the sequence about the calls?

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


    function setup() {
            noCanvas();
			wordGIF()
				.then(results => {
					createP(result.word); 
					createImg(result.img);	
                    return wordGIF();
				}).then(results => {
					createP(result.word); 
					createImg(result.img);	
                })
				.catch(err => console.log(err));
    }

    async function wordGIF(num) {
			let response1 =  await fetch(wordnikAPI);
			let json1 = await response.json();
			let word = json1.word;
			let response2 = await fetch(giphyAPI + json1.word);
			let json2 = await response2.json();
			let img = json2.data[0].images['fixed_height_small'].url;
			return {
				word: json1.word,
				img: img_url
			}
    }
 
Right now we just chained promises in the **setup()** function. Now lets say I put **num** as an argument for the **wordGIF()** function which means the number of letters that I want. It would give me words that correspond to the number of letters inputted.


Now if you run the program, you'll notice the chained promises will display two words and their two corresponding pictures. Now what if we didnt chain the promises.


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


    function setup() {
            noCanvas();
			wordGIF(4)
				.then(results => {
					createP(result.word); 
					createImg(result.img);	
                    return wordGIF();
				})
				.catch(err => console.log(err));

            wordGIF(5)
				.then(results => {
					createP(result.word); 
					createImg(result.img);	
                    return wordGIF();
				})
				.catch(err => console.log(err));    
    }

    async function wordGIF(num) {
			let response1 =  await fetch(wordnikAPI);
			let json1 = await response.json();
			let word = json1.word;
			let response2 = await fetch(giphyAPI + json1.word);
			let json2 = await response2.json();
			let img = json2.data[0].images['fixed_height_small'].url;
			return {
				word: json1.word,
				img: img_url
			}
    }
 
Now we have two promises running in parallel and when they both come back, create the word and corresponding image. When running the program, you'll see that it works. When they're happening in parallel, we're not really sure about the order of execution. One way to deal with that is to chain them or wait till everything is done and then show the results and thats where **promise.all** comes into play. **Promise.all** requires and array.


    function setup() {
            noCanvas();

            let promises = [];
            for (let i = 0; i < 100; i++) {
                promises.push(wordGIF(4));
            }


            let promises = [wordGIF(3), wordGIF(4), wordGIF(5)];
            Promise.all(promises)
                .then((results) => {
                    for(let i = 0; i < results.length; i++) {
                        createP(results[i].word);
                        createImg(results[i].img);
                    }
                })
                .catch((err) => console.log(err));
    }

    async function wordGIF(num) {
			let response1 =  await fetch(wordnikAPI); 
			let json1 = await response.json();
			let word = json1.word;
			let response2 = await fetch(giphyAPI + json1.word);
			let json2 = await response2.json();
			let img = json2.data[0].images['fixed_height_small'].url;
			return {
				word: json1.word,
				img: img_url
			}
    }


This is the skeleton. The idea is if I create an array of three promises. I can say "when all promises are completed and result, give me the result of all those promises in an array in the same order as the original problem." 

**Promise.all** maybe not be the best solution sometimes because it *all or nothing*. If any of the promises are rejected, then we don't get any results.  



# Using *try* and *catch* with Promises

To counter the problem using **Promise.all** is using the **try** and **catch** keywords. 

    function setup() {
            noCanvas();

            let promises = [];
            for (let i = 0; i < 100; i++) {
                promises.push(wordGIF(4));
            }


            let promises = [wordGIF(3), wordGIF(4), wordGIF(5)];
            Promise.all(promises)
                .then((results) => {
                    for(let i = 0; i < results.length; i++) {
                        createP(results[i].word);
                        if (results[i].img !== null) {
                            createImg(results[i].img);
                        }
                    }
                })
                .catch((err) => console.log(err));
    }

    async function wordGIF(num) {
			let response1 =  await fetch(wordnikAPI); 
			let json1 = await response.json();
			let word = json1.word;
			let response2 = await fetch(giphyAPI + json1.word);
			let json2 = await response2.json();
            let img_url = null;

            try {
			    let img_url = json2.data[0].images['fixed_height_small'].url;
            }
            catch (err) {
                console.log("No image found for " + json1.word);
                console.error(err);
            }

			return {
				word: json1.word,
				img: img_url
			}
    }

In our async function, let first try making our **img_url** variable equal to null then **try** to get the **img_url** from the data. It might not exist but thats okay, lets **try**. Then we'll return **json1.word** and it could be null if it doesnt work.

