# Storage methods for the browser

#### Useful references

- [Overview of techniques](http://www.html5rocks.com/en/tutorials/offline/storage/)
- [Research on quotas](http://www.html5rocks.com/en/tutorials/offline/quota-research/)
- [Demo page](http://demo.agektmr.com/storage/)

#### Allowed file sizes

Since we are trying to save an entire music library in the browser, we need the ability to save at least 1.000 files with up to 10 MB per file. The table below shows the possible techniques of saving data with the maximum total size.

Technique       | Chrome (Desktop)      | Chrome (Andriod)
--------------- | --------------------- | ---------------------
FileSystem      | Up to requested quota | Up to requested quota
IndexedDB       | Up to requested quota | Up to requested quota
WebSQL          | Up to requested quota | Up to requested quota
LocalStorage    | 10 MB                 | 10 MB
SessionStorage  | 10 MB                 | 10 MB

As shown in the table, LocalStorage and SessionStorage are not possible for our use-case, and we will concentrate on the last 3 options.

#### Browser Support

Since we are writing our application with a main focus on Chrome (Desktop & Android), we didn't make browser support to one of our main decision points, but it is an interesting discussion point nevertheless. 

Technique       | Browser Support
--------------- | ---------------------
FileSystem      | [Chrome, Opera](http://caniuse.com/#search=FileSystem) - no longer maintained
IndexedDB       | [All major browsers](http://caniuse.com/#search=IndexedDB), but not supported in Chrome for iOS
WebSQL          | [Chrome, Safari, Opera](http://caniuse.com/#search=WebSQL)- no longer maintained

#### Performance

Using the [demo page](http://demo.agektmr.com/storage/), we tested the performance of the different techniques for our use-case (multiple hundred files of about 5-10MB).

Technique       | Comment
--------------- | ---------------------
FileSystem      | Fast, handles even big amounts of data well
IndexedDB       | Freezes the browser, really slow, non-predictable behaviour with permanent and temporary storage (might be a bug in the demo page) 
WebSQL          | Freezes the browser, slow, doesn't handle larger files well, crashes the browser easily

#### Decision for mp3: FileSystem

Although the FileSystem API is no longer officially maintained, it is still the fastest and most performant method of storing large amounts of data, without any competition. The missing availability on other browsers as a drawback was okay for us, since the application focuses on Chrome for Desktop and Andriod.

#### Decision for metadata: localStorage

We had to decide between localStorage or [level.js](https://github.com/Level/level-browserify) (build on IndexedDB) for storing our browser-side metadata. Side-by-side comparsion:

Topic | localStorage | level.js
------|--------------|-------------
API | easy, syncronous | abstract, callbacks, async
Storage | 10MB | Quota
Speed | ~4ms per key read/write pair | ~250ms per key read/write pair

Because of the additional abstraction and relatively slow performance of level.js, we decided for localStorage for our use case. It is a lot faster, and we will probably not reach the memory limitations saving simple text metadata.
