<script lang="ts">
  import songs from "../lib/songs.json";

  import Fa from "svelte-fa";
  import {
    faCaretLeft,
    faCaretRight,
    faPause,
    faPlay,
  } from "@fortawesome/free-solid-svg-icons";

  import { onMount } from "svelte";
  import YouTubePlayer from "youtube-player";
  import type { YouTubePlayer as PlayerType } from "youtube-player/dist/types";

  const _id = (Math.random() + Date.now()).toString(8);

  let curr = 0;
  let ytplayer: PlayerType;
  let playerState;
  let playing = false;

  let hideBanner = false;
  let bannerTransition = false;

  function nextVideo() {
    curr = (curr + 1) % songs.length;

    loadVideo(ytplayer);
  }

  function prevVideo() {
    curr = (curr - 1 + songs.length) % songs.length;
    loadVideo(ytplayer);
  }

  async function loadVideo(player: PlayerType) {
    console.log(curr);
    const song = songs[curr];
    const id = song.url.split("=")[1];
    if ((await player.getPlayerState()) == 1) {
      playing = true;
    } else {
      playing = false;
    }
    await player.loadVideoById(id);
    if (playing) {
      player.playVideo();
    } else {
      player.pauseVideo();
    }
  }

  onMount(() => {
    let player = YouTubePlayer(_id, {
      playerVars: {
        //@ts-ignore
        autoplay: 1,
        controls: 0,
        loop: 1,
        //@ts-ignore
        color: "black",
        //@ts-ignore
        modestbranding: true,
        showinfo: 0,
        iv_load_policy: 3,
        playsinline: 1,
        rel: 0,
        ecver: 2,
      },
    });
    ytplayer = player;
    ytplayer.on("stateChange", (e) => {
      playerState = e.data;
    });
    loadVideo(ytplayer);
  });
</script>

<div class="vignette" />
{#if !hideBanner}
  <div
    class="z-30 absolute left-0 top-0"
    style={bannerTransition
      ? "animation: fade-out 1s ease-in-out; animation-iteration-count: 1;"
      : ""}
  >
    <div
      class="w-screen h-screen from-zinc-10 bg-black bg-[url(/lofiwallpaper.jpg)] bg-cover brightness-50 absolute"
    />
    <button
      class="w-screen h-screen text-2xl absolute"
      on:click={() => {
        bannerTransition = true;
        setTimeout(() => {
          hideBanner = true;
        }, 900);

        ytplayer.playVideo();
      }}
    >
      Click anywhere to start the lofi player!
    </button>
  </div>
{/if}
<div class="h-screen w-screen overflow-hidden -z-10 brightness-75 absolute">
  <div
    id={_id}
    class="pointer-events-none absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 h-[200vh] w-screen"
  />
</div>
<div
  class="flex flex-col gap-4 mt-4 absolute bottom-0 left-0 p-4 bg-black/25 rounded-tr-3xl z-20"
>
  <div class="flex flex-col pr-2">
    <h2 class="flex items-center gap-2">
      <span class="text-lg">{songs[curr].author}</span>
      <span class="">- {songs[curr].name}</span>
    </h2>
  </div>
  <div class="flex gap-4">
    <button
      class="control-button p-3"
      on:click={() => {
        prevVideo();
      }}><Fa icon={faCaretLeft} /></button
    >

    {#if playing}
      <button
        class="control-button p-4"
        on:click={() => {
          playing = false;
          ytplayer.pauseVideo();
        }}><Fa icon={faPause} /></button
      >
    {:else}
      <button
        class="control-button p-4"
        on:click={() => {
          playing = true;
          ytplayer.playVideo();
        }}><Fa icon={faPlay} /></button
      >
    {/if}

    <button
      class="control-button p-3"
      on:click={() => {
        nextVideo();
      }}><Fa icon={faCaretRight} /></button
    >
  </div>
</div>
