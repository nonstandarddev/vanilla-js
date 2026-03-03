# Vanilla JS - Project Guidelines

This repository describes a set of &amp; principles for building 'Vanilla JS' applications.

# Principles

## One Big File

Perhaps counter to expectations, one approach to structuring basic webpages is to organise all
of one's JavaScript into *one big file* (which we name something like `index.js`).

I found this recommendation in an interesting blog post on the 'Plain English' site which you are free
to peruse [here](https://plainenglish.io/blog/the-basic-vanilla-js-project-setup-9290dce6403f).

Whilst I remain to be fully convinced, of note, the author provides a few reasons for this,

* First, refusing to 'modularise' one's code does *not* necessarily imply a lack of structure. You
are still free to structure and organise the code in one file (and you absolutely should)
* Second, web development is different to other software development scenarios in that browsers
are not natively designed to deal with 'modules' and therefore require a separate *build tool* (such
as `webpack`). This can add a burden to CI/CD and other deployment workflows that is otherwise
completely unnecessary when only one file is present
* Third, modern code editors are optimised to provide efficient 'lookup' operations on one file. They
are also almost always equipped with an 'outline' view that provides an overview of all of the sections
within one script (aiding navigation all the same)
* Modularisation has something in common with 'structuring' one's code but they are technically
distinct objectives: modules are supposed to encapsulate a central 'idea' or, more concretely, a
single 'entrypoint' which is enabled by a host of supplementary functions that are *private* to 
that module (and are only consistent with the purpose of that module). Otherwise, we would describe
the design as flawed (and largely incoherent)

## Layout

The proposed layout for this single `index.js` file is as follows,

* Constants, helpers
* State (or store)
* State accessor functions (setters &amp; getters)
* DOM node *references*
* DOM update functions
* Event handlers
* Event handler *bindings*
* Initialisation

## Implementation

Here is the rough implementation,

```js
/*
    1: Constants & helpers
*/
const TODAY = Date.now();
const LOADING = 0, READY = 1, ERROR = 2;
...

/* 
    2: State
    
    NB: may be defined in one or more variables
*/
let state = { ... };

/* 
    3: State Manipulation (Accessors)

    NB: includes the 'getters' and 'setters' associated with backend services
*/

// Setters
let setLoading = () => state.view = LOADING;
let setReady = () => state.view = READY;
...

// Getters
let isLoading = () => state.view === LOADING;
let isReady = () => state.view === READY;
...
let loadSongs = () => fetch('/api/songs/')
  .then(res => res.json())
  .then(data => {
    state.songs = data.songs
    state.view = READY
  })
  .catch(() => state.view = ERROR);

/*
    4: DOM References

    Alternatives include something like the following,

    let REFS = {
        playback: {
            play: document.getElementById('play'),
            stop: document.getElementById('stop'),
        },
        views: {
            [LOADING]: document.getElementById('play'),
            [READY]: document.getElementById('loading'),
        },
    }

*/

let D = document;

// DOM nodes (individual)
let $play = D.getElementById('play');
let $stop = D.getElementById('stop');
let $viewLoading = D.getElementById('view-loading');
let $viewReady = D.getElementById('view-ready');
...

// DOM nodes (series)
let $$instruments = D.querySelectorAll('.instrument-option');
...

/* 
    5: DOM Updates (i.e. logic for 'modifying' the DOM in certain detailed ways)
*/

let updateView = () => {
  $viewLoading.classList.toggle('hidden', !isLoading());
  $viewReady.classList.toggle('hidden', !isReady());
  $viewError.classList.toggle('hidden', !isError());
};

let updateSongDetails = (songData, index) => {
  let $song = $$songs[index];
  $song.querySelector('.active')
    .classList.toggle('hidden', !isSelected(index));
  $song.querySelector('.title').textContent = songData.title;
  $song.querySelector('.tempo').textContent = songData.tempo + 'bpm';
};

let updateSongs = () => state.songs.forEach(updateSongDetails);
...

/*
    6: Event Handlers

    NB: these are called by event listeners that listen to DOM (and other events e.g. WebSockets)
*/

let onPlay = () => {

  setPlay();
  updatePlaybackButton();
  updateScoresheet();
  startPlaybackTimer();
};

let onStop = () => {

  stopPlaybackTimer();
  setStop();
  updatePlaybackButton();
  updateScoresheet();
};

let onEdit = scores => {

  onStop();
  setScores(scores);
  updateScoresheet();
};

let onSongLoaded = () => {

  setInitialScores();
  updateScoresheet();
  updatePlaybackButton();
  updateLoadError();
};

let onLoadSong = songId => {

  onStop();
  loadSong(songId).then(onSongLoaded);
};
...

/*
    7: Event Bindings
*/
$play.onclick = () => onPlay();
$stop.onclick = () => onStop();
$scoreEditor.oninput = ev => onEdit(ev.target.value);
...

/*
    8: Initialisation
*/
setReady();
updateView();

```