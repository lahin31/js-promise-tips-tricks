# JavaScript Promises Tips & Tricks

## Tip 1: Promise Chaining

instead of this, 

```js
axios.get('https://jsonplaceholder.typicode.com/posts')
    .then(res => {
        console.log(res.data);
        getPhotos()
            .then(photos => {
                console.log(photos.data)
            })
    })
    .catch(err => {
        console.error(err)
    })

function getPhotos() {
    return axios.get('https://jsonplaceholder.typicode.com/photos');
}
```

use like this,

```js
axios.get('https://jsonplaceholder.typicode.com/posts')
    .then(res => {
        console.log(res.data);
        return getPhotos();
    })
    .then(res => {
        console.log(res.data);
    })
    .catch(err => {
        console.error(err)
    })

function getPhotos() {
    return axios.get('https://jsonplaceholder.typicode.com/photos');
}
```

## Tip 2: Speed up promises with promise.all()

instead of this,

```js
function getUsersLength() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(['lahin', 'hussain'].length)
        }, 5000);
    })
}

function getPhotosLength() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(['photo 1', 'photo 2'].length)
        }, 6000);
    })
}

async function doSomething() {
    try {
        const usersLength = await getUsersLength();
        const photosLength = await getPhotosLength();
    
        console.log(usersLength, photosLength); // 2, 2
    } catch(err) {
        throw new Error(err);
    }
}

doSomething();
```
after 11 seconds you can see the result 2, 2.

use like this,

```js
function getUsersLength() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(['lahin', 'hussain'].length)
        }, 5000);
    })
}

function getPhotosLength() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(['photo 1', 'photo 2'].length)
        }, 6000);
    })
}

async function doSomething() {
    try {
        const [usersLength, photosLength] = await Promise.all([getUsersLength(), getPhotosLength()]);
        console.log(usersLength, photosLength); // 2, 2
    } catch(err) {
        throw new Error(err);
    }
}

doSomething();
```
`getUsersLength()` took 5 seconds to resolve and `getPhotosLength()` took 6 seconds to resolve. `promise.all()` will executes those functions parallelly, so after 6 seconds you can see the result.

<p align="center">
  <img src="./assets/images/promise-all-1.png" alt="promise.all">
</p>

## Tip 3: await inside for...of and foreach loop

instead of this,

```js
function getUsersTitle() {
    return axios.get('https://jsonplaceholder.typicode.com/users')
        .then(res => res.data.map(({ name }) => name));
}

function makeUppercase(val) {
    return Promise.resolve(val.toUpperCase());
}

async function doSomething() {
    const names = await getUsersTitle();

    names.forEach(async name => {
        const uppercasename = await makeUppercase(name)
        console.log(uppercasename);
    });

    console.log("Finished");
}

doSomething();
// Finished
// LEANNE GRAHAM
// ERVIN HOWELL
// CLEMENTINE BAUCH
// PATRICIA LEBSACK
// CHELSEY DIETRICH
// MRS. DENNIS SCHULIST
// KURTIS WEISSNAT
// NICHOLAS RUNOLFSDOTTIR V
// GLENNA REICHERT
// CLEMENTINA DUBUQUE
```

forEach loop can't process data in sequence.

use for..of to get data in sequence,

```js
function getUsersTitle() {
    return axios.get('https://jsonplaceholder.typicode.com/users')
        .then(res => res.data.map(({ name }) => name));
}

function makeUppercase(val) {
    return Promise.resolve(val.toUpperCase());
}

async function doSomething() {
    const names = await getUsersTitle();

    for(let name of names) {
        const uppercasename = await makeUppercase(name)
        console.log(uppercasename)
    }

    console.log("Finished");
}

doSomething();
// LEANNE GRAHAM
// ERVIN HOWELL
// CLEMENTINE BAUCH
// PATRICIA LEBSACK
// CHELSEY DIETRICH
// MRS. DENNIS SCHULIST
// KURTIS WEISSNAT
// NICHOLAS RUNOLFSDOTTIR V
// GLENNA REICHERT
// CLEMENTINA DUBUQUE
// Finished
```
