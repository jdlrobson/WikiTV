<template>
    <div class="show">
        <h2 class="show-heading">{{ title }}</h2>
        <div class="show-content">
            <img v-if="thumbnail" :src="thumbnail.source">
            <p>{{ extract }}</p>
        </div>
        <video class="show-video" v-if="videoUrl" autoplay>
            <source :src="`${videoUrl}`" :type="videoType">
        </video>
        <video class="show-video" v-if="videoUrl" autoplay>
            <source :src="`${videoUrl}#t=50`" :type="videoType">
        </video>
        <audio v-if="audioUrl" autoplay>
            <source :src="`${audioUrl}`" :type="audioType">
        </audio>
        <audio v-if="audioUrl" autoplay>
            <source :src="`${audioUrl}#t=50`" :type="audioType">
        </audio>
        <a target="_blank"
            :href="url"
            class="show-static"></a>
    </div>
</template>
<script>
import {  defineComponent } from 'vue';
export default defineComponent( {
    props: {
        page: {
            type: Object
        }
    },
    setup( props ) {
        const { title, thumbnail, extract, audio, video } = props.page;
        const audioUrl = audio[0];
        const videoUrl = video[0];
        const videoType = videoUrl ? `video/${videoUrl.split('.').slice(-1)}` : '';
        const audioType = audioUrl ? `audio/${audioUrl.split('.').slice(-1)}` : '';
        return {
            url: `https://en.wikipedia.org/wiki/${title}`,
            videoUrl,
            videoType,
            audioUrl,
            audioType,
            title, thumbnail, extract
        };
    }
} );
</script>
<style>
.show-heading {
    margin: 0;
}
.show {
    height: 100%;
    padding: 35px 20px;
    color: white;
    position: relative;
    overflow: hidden;
}

.show-static {
    opacity: 0.9;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background:
    linear-gradient(to bottom, transparent, #aaa4, #8881, #6664, #4445, #2228, #4443, transparent),
    repeating-linear-gradient(transparent 0 2px, #25242950 2px 4px);
    animation: moveBand 8s linear infinite;
}

.show-content {
    display: flex;
}
.show-content img {
    width: 30%;
    min-width: 30%;
    min-height: 100px;
    background: gray;
    object-fit: contain;
    margin-right: 20px;
    max-height: 200px;
}

@keyframes moveBand {
  0% { background-position-y: 0, 0; }
  100% { background-position-y: -221px, -150px; }
}

.show-video {
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    right: 0;
    opacity: 0.4;
}
</style>