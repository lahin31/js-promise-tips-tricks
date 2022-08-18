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
## Tip 4: Always better to return promise

instead of this,

```js
function getUsers() {
    return axios.get('https://jsonplaceholder.typicode.com/users');
}

function getPhotos() {
    return axios.get('https://jsonplaceholder.typicode.com/photos'); // 5000 photos
}

getUsers()
    .then(res => {
        getPhotos()
            .then(photos => {
                console.log(photos.data)
            })
    })
    .then(() => {
        console.log("JS Promise");
    })
    .catch(err => {
        console.error(err);
    })
```

You can see it prints `JS Promise` first then prints Photos array, this can be problematic. Use like this,

```js
function getUsers() {
    return axios.get('https://jsonplaceholder.typicode.com/users');
}

function getPhotos() {
    return axios.get('https://jsonplaceholder.typicode.com/photos'); // 5000 photos
}

getUsers()
    .then(res => {
        return getPhotos();
    })
    .then(photos => {
        console.log(photos.data);
    })
    .then(() => {
        console.log("JS Promise");
    })
    .catch(err => {
        console.error(err);
    })
```

Now you can see it prints Photos array first then it prints `JS Promise` last.

## Tip 5: Cancelling a promise (react)

```jsx
useEffect(() => {
    const controller = new AbortController();

    axios.get('/{{api_endpoint}}', {
        signal: controller.signal
    })
        .then(res => {
            console.log(res);
        });

    return () => controller.abort();
}, []);
```

## Tip 6: Debounce an API call with promise (react)

```jsx
const [name, setName] = useState('');
const [suggestions, setSuggestions] = useState([]);

const handleNameChange = e => {
    fetch(`${api}/search?q={e.target.value}`)
        .then(res => res.json())
        .then(res => setSuggestions(res.data.items));
}

const debounce = func => {
    let timer;
    return function(...args) {
        const context = this;
        if(timer) clearTimeout(timer);
        timer = setTimeout(() => {
            timer = null;
            func.apply(context, args);
        }, 500)
    }
}

const optimizedFunc = useCallback(debounce(handleNameChange));
```
