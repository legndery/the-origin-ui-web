---
title:  "Loops, Asynchronous functions and Returning Collective Results in Node.js (with bonus trivia at the end)"
date:   2021-12-15 02:45:54 +0530
categories: Javascript
classes: wide
author: Aritra
---
I like to solve stackoverflow questions, even sometimes like to browse the node.js tag for relevant questions. One of the most common question pattern is:
>Hey, I am looping and doing some HTTP requests or Model queries for each iteration and pushing it to my results variable but It is BLANK when I print it. What the Heck! Help!

Let’s see a piece of code. Using request module:
{% highlight javascript %}
const element = ['element1', 'element2', 'element3']
const results = [];
for(let i=0;i<element.length;i++){ //or forEach
    const el = element[i];
    request(`url?el=${el}`, (err,resp, body)=>{
        results.push(body);
    });
}
console.log(results);
{% endhighlight %}

Get it? **Get it**? No? OK. Lemme give you another chance, using Promises while querying data:
{% highlight js %}
const products = Products.find();
const results = [];
products.forEach((product)=>{
    Price.find({ where: { ProductID: product.id}})
    .then((data)=>{
        results.push(data);
    })
});
console.log(results);
{% endhighlight %}

You get the picture right? *right?*

Okay. Let me explain:
## Why is the variable undefined/blank ? It doesn’t make any sense
>~OH you do? good for you. Go find another post, this isn’t for you, You Smartass! *Proceeds to cry in a corner*~

So, Basically there are 2 simple informations you need to know,
1. loops are synchronous. Means it will wait for the current operation to complete and go to the next one.
1. And asynchronous operations like Network I/O like HTTP request, or File I/O they are non blocking, that means it will INITIATE the request and return the control. It never blocks the thread that is running the program.
![My probably not helpful screenshot](/assets/images/posts/loop-async.jpeg)

>Now, let’s piece these two together, loop executes the async operations one by one, but as they are non blocking the control is immediately returned to the loop and it executes it’s next iteration and so on. At the end when you print the result, it is blank. Because all the async operations are initiated alright, but they are not yet completed.
{% comment %}
Got it? Now, Let’s take a real life imaginary (oxymoron much?) scenario:

**Boss:** Hey, call these 2 numbers and check if their spouse wants a credit card to buy random expensive stuff plunging themselves into a soul-crushing debt.  
**You**: Okay. Will do boss.  
— Proceeds to calling those numbers  
**You**: Hey, can you check if your spouse wants a credit card to buy random expensive stuff plunging themselves into a soul-crushing debt?  
**Caller1**: Umm.. ok.. Let me check with her. I will call you after 5mins.  
— Proceeds to calling the next number  
**You**: Hey, can you check if your spouse wants a credit card to buy random expensive stuff plunging themselves into a soul-crushing debt?  
**Caller2**: Okay! Don’t know, need to ask him, will notify you after an hour.  
**You**: Boss Nobody wants credit card.
— A FEW MOMENTS LATER (Please read in Spongebob voice)  
**Boss:** So, did anyone needed a credit card? huh?

# You Died.
Yes. Because no caller returned your call YET but as your boss asked you tried to remember a non existent memory, your brain had internal haemorrhage and you died. Rest in peace.
{% endcomment %}

Now that hopefully understand what is happening, let’s quickly see some code examples on how to handle this, I will use a dummy HTTP request to http://dummy.restapiexample.com/ using request module using callbacks and axios module which supports Promises. Feel free to Promisify callback based modules based on your needs.  
BUT Those of you like callbacks and not promises, START USING PROMISES.

{% highlight js %}
const request = require('request');

const empIds = ['72632', '72633', '72634'];
const dummyRESTurl = 'http://dummy.restapiexample.com/api/v1/employee/';
const result = [];

let count = 0;
empIds.forEach((empID, i)=>{
    request(`${dummyRESTurl}${empID}`, (err, resp, body)=>{
        result.push(JSON.parse(body));
        count++;
        if(count === empIds.length){
            render();
        }
    })
})
function render(){
    //does something with the collective data
    console.log(result);
}
{% endhighlight %}
Now, let’s use some promises. Here I am using Promise.all which waits for all the promises to resolve.
{% highlight js %}
const axios = require('axios');

const empIds = ['72632', '72633', '72634'];
const dummyRESTurl = 'http://dummy.restapiexample.com/api/v1/employee/';

let promises = [];
empIds.forEach((empID, i)=>{
    promises.push(axios.get(`${dummyRESTurl}${empID}`));
})

Promise.all(promises)
.then((result)=>{
    //do something with data
    console.log(result.map(r=>r.data));//because axios returns full response
})
{% endhighlight %}

Now, let’s use some ES6 async-await . With async-await you can do serial as well as parallel execution of your async functions with ease.
{% highlight js %}
const axios = require('axios');

const empIds = ['72632', '72633', '72634'];
const dummyRESTurl = 'http://dummy.restapiexample.com/api/v1/employee/';
const result = [];

//Serial
(async () => {
    for (let i = 0; i < empIds.length; i++) {
        const body = await axios.get(`${dummyRESTurl}${empIds[i]}`);
        result.push(body.data);
    }
    console.log(result);
})()

//Parallel
const promises = [];
empIds.forEach((empID) => {
    promises.push(axios.get(`${dummyRESTurl}${empID}`));
});
(async () => {
    const pResult = await Promise.all(promises);
    console.log(pResult.map(r => r.data))
})();
{% endhighlight %}

-------- 
Bonus Knowledge:

While using async-await you cannot use Array.forEach/ map/ reduce because it will not wait for the promise to finish. If we want to explore what Array.forEach is, it will be something like:
{% highlight js %}
function forEach(arr, callback){
    for(let i=0; i<arr.length; i++){
        callback(arr[i], i, arr);
    }
}
{% endhighlight %}
as you can see It doesn’t support async function. Even if the callback is marked as async, forEach doesn’t await the callback. To make one custom forEach it will be something like:
{% highlight js %}
function forEachAsync(arr, callback){
    for(let i=0;i<arr.length;i++){
        await callback(arr[i], i, arr);
    }
}
{% endhighlight %}

So, hope it helped you.. or not.. if you are reading this line, then Thank you for reading it all the way :)