<template>
  <div class="app">
    <div :class="tvClass">
      <div class="tv-body">
        <div class="tv-screen">
          <show v-if="pages[ index ]" :page="pages[ index ]" :key="index">
          </show>
          <div class="tv-screen-pause" v-if="isPaused">
            <cdx-icon :icon="cdxIconPause"></cdx-icon>
          </div>
        </div>
        <div class="tv-controls">
          <button class="tv-on-switch" @click="toggleOnOff">on</button>
          <button @click="rewind">rewind</button>
          <button v-if="!isPaused" @click="togglePause">pause</button><button @click="togglePause" v-else>play</button>
          <button @click="fastForward">forward</button>
          <dial @click="changeChannel" :defaultValue="channelNumber"></dial>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Dial from './Dial.vue';
// https://commons.wikimedia.org/wiki/File:Mechanical_tack.ogg
import tack from './tack.ogg';
import soundlogo from './soundlogo.wav';
import { CdxIcon	} from '@wikimedia/codex';
import { cdxIconPause	} from '@wikimedia/codex-icons';
import {  defineComponent, ref, onMounted } from 'vue';
import Show from './Show.vue';
import md5 from 'blueimp-md5';

const index = ref( -1 );
const pages = ref( [] );
const interval = ref( 0 );

const isPaused = ref( false );
const isOn = ref( false );
const audio = new Audio( tack );
const onOffAudio = new Audio( soundlogo );
let cache = [];
let CHANNEL = 'popular';
/*try {
  cache = JSON.parse( localStorage.getItem( `wikitv-cache-${CHANNEL}` ) || '[]' )
} catch ( e ) {
  cache = [];
}*/
const CHANNEL_MOST_READ = {
  _url: 'https://en.wikipedia.org/w/api.php?origin=*&action=query&format=json&prop=images%7Cpageimages&titles=United%20States&generator=mostviewed&formatversion=2&imlimit=max&gpvimmetric=pageviews&gpvimlimit=10',
  url() {
    return this._url;
  },
  next(result) {
    const continueOptions = result.continue;
    const nextUrl = new URL( this._url );
    Object.keys(continueOptions).forEach((key) => {
      nextUrl.searchParams.set( key, continueOptions[key] );
    })
    this._url = nextUrl.toString();
  }
};
const CHANNEL_NEARBY = {
  url() {
    return `https://en.wikipedia.org/w/api.php?action=query&format=json&origin=*&prop=images%7Ccoordinates&titles=United%20States&generator=geosearch&formatversion=2&colimit=20&imlimit=20&ggsradius=10000&ggslimit=20&ggsnamespace=0&ggscoord=${playerLocation.join('|')}`;
  },
  next(result) {
    // jump to a new location
    playerLocation = [ playerLocation[ 0 ] + 0.008983, playerLocation[1] +  0.015060 ];
  }
};

const channels = [
  CHANNEL_MOST_READ,
  CHANNEL_NEARBY
];
let channelNumber = ref( 0 );
const getShow = () => {
  const channel = channels[channelNumber.value];
  if ( !channel ) {
    return;
  }
  if ( cache.length === 0 ) {
    return fetch(channel.url())
      .then((r) =>r.json()).then((results) => {
        const pages = results.query.pages.filter((p) => p.ns === 0 && p.title !== 'Main Page');
        localStorage.setItem(  `wikitv-cache-${CHANNEL}`, JSON.stringify( pages ) )
        cache = cache.concat( pages );
        channel.next(results);
        return getShow();
      } );
  } else {
    return Promise.resolve( cache.shift() );
  }
};

let playerLocation = [];
const summaries = {};
const getSummary = ( title, obj ) => {
  if ( summaries[title] ) {
    return Promise.resolve( summaries[title] );
  }
  return fetch(`https://en.wikipedia.org/api/rest_v1/page/summary/${title}`)
    .then((r) =>r.json())
    .then(( {  title, thumbnail, extract } ) => {
      summaries[title] = Object.assign( {  title, thumbnail, extract }, obj );
      return getSummary( title );
    })
};


function toUrlFromTitle(file) {
  file = file.replace(/^File:/, '').replace(/\s+/g, '_');
  const encoded = encodeURIComponent(file);
  const base = 'https://upload.wikimedia.org/wikipedia/commons';
  const hash = md5(file);
  const ns = `${hash[0]}/${hash[0]}${hash[1]}`;
  return `${base}/${ns}/${encoded}`;
}

const load = () => {
  getShow()
    .then(( page ) => {
      const { title, images } = page;
      const extension = ( name ) => name.toLowerCase().split('.').slice(-1)[0] || '';
      const toUrl = ( page ) => toUrlFromTitle( page.title )
        
      const audio = ( images || [] )
        .filter((name) => ['ogg', 'wav'].includes(extension(name.title)))
        .map(toUrl);
      const video = ( images || [] )
        .filter((name) => ['webm'].includes(extension(name.title)))
        .map(toUrl);
      const ext = new Set(
        ( images || [] ).map((p) => extension(p.title))
      );
      console.log(title, ext);
      getSummary( title, { audio, video } )
        .then(( newPage ) => {
          index.value = index.value + 1;
          pages.value = pages.value.concat( newPage );
        })
    })
};

const surf = () => {
  if ( interval.value ) {
    clearInterval( interval.value );
  }
  interval.value = setInterval(() => {
    if ( !isPaused.value && isOn.value ) {
      load();
    }
  }, 7000 );
};

export default defineComponent( {
  components: {
    Dial,
    Show,
    CdxIcon
  },
  computed: {
    tvClass() {
      return {
        tv: true,
        'tv-off': !isOn.value
      };
    }
  },
  methods: {
    changeChannel() {
      channelNumber.value = channelNumber.value + 1;
      if ( channelNumber.value > channels.length - 1 ) {
        channelNumber.value = 0;
      }
      cache = [];
    },
    toggleOnOff() {
      onOffAudio.play();
      isOn.value = !isOn.value;
      if ( isOn.value ) {
        const start = () => {
          load();
          surf();
        }
        navigator.geolocation.getCurrentPosition((position) => {
          playerLocation = [ position.coords.latitude, position.coords.longitude ];
          channelNumber.value = 1;
          start();
        }, () => {
          start();
        });
      }
    },
    togglePause() {
      audio.play();
      isPaused.value = !isPaused.value;
    },
    rewind() {
      if ( index.value > 0 ) {
        audio.play();
        index.value = index.value - 1;
        surf();
      }
    },
    fastForward() {
      if ( index.value < pages.value.length - 1 ) {
        audio.play();
        index.value = index.value + 1;
        surf();
      }
    }
  },
  setup() {
    if ( isOn.value ) {
      load();
      surf();
    }
    return {
      channelNumber,
      cdxIconPause,
      isPaused,
      index,
      pages
    }
  }
} );
</script>
<style>
.tv * {
  box-sizing: border-box;
}
.tv {
  --width: 800px;
  height: calc( var( --width ) * 2/3 );
  width: var( --width );
  border: solid 1px black;
  padding: 15px;
  border-radius: 20px;
  margin: auto;
  background: gold;
  box-sizing: border-box;
  overflow: hidden;
}
.tv-body {
  display: flex;
  border: solid 1px black;
  padding: 20px;
  background: black;
  height: 100%;
  align-items: center;
  gap: 20px;
}
.tv-screen {
  position: relative;
  border-radius: 85px 85px 85px 85px / 100px 100px 100px 100px;
  overflow: hidden;
  flex-grow: 1;
  width: 100%;
  height: 100%;
  box-shadow: 0 0 5px 2px #111 inset;
  background: #302d30;
  padding-bottom: 20px;
  transition: width, height 0.5s ease-in;
  width: 100%;
}
.tv-off .tv-screen {
  height: 1px;
  padding: 0px;
}
.tv-controls {
  width: 100px;
  display: flex;
  flex-flow: column;
  padding-left: 15px;
  justify-content: center;
  padding: 15px;
  background: gold;
  align-items: center;
  box-shadow: -2px 2px 6px #111 inset, -1px 2px 3px #aaa;
  height: 200px;

}

.tv-on-switch {
  cursor: pointer;
  background: gray;
  border-radius: 25px;
  color: white;
  height: 25px;
  border: 0;
  margin-bottom: 40px;
  border: solid 5px black;
}
.dial {
  margin-top: 10px;
}
.dial:before {
  content: '.';
  width: 5px;
  height: 5px;
  border-radius: 5px;
}
.tv-screen-pause {
    color: white;
    display: flex;
    margin: auto;
    opacity: 0.2;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    align-items: center;
    justify-content: center;
}
.tv-screen-pause .cdx-icon {
  color: white;
  width: 100px;
  height: 100px;
}
</style>