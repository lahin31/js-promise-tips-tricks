# JavaScript Promises Tips & Tricks

## Tips #1: Promise Chaining

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