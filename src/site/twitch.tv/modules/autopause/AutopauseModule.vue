<template />

<script setup lang="ts">
import { log } from "@/common/Logger";
import { declareModule } from "@/composable/useModule";
import { declareConfig, useConfig } from "@/composable/useSettings";
log.info('run;')

const { markAsReady } = declareModule("autopause", {
	name: "Autopause",
   depends_on: [],
});

const isEnabled = useConfig<boolean>("general.autopause");


function attemptPause():boolean {
   var player:HTMLVideoElement | null = document.querySelector('.video-player__container video');
   if (player == null) return false;
   player.pause();
   player.onplaying = () => {
      if (!player) return;
      player.pause();
      player.onplaying = null;
   };
   return true;
}

if (isEnabled.value) {
   log.info('enabled');
   if (!attemptPause()) {
      // if the video element doesnt exist yet, try again until it does
      log.info('fail');
      var intervalId:number;
      intervalId = setInterval(() => {
         if (attemptPause()) clearInterval(intervalId);
      }, 100);
   }
}

markAsReady();
</script>

<script lang="ts">
export const config = [
	declareConfig("general.autopause", "TOGGLE", {
		label: "Autopause On Frontpage",
		hint: "Automatically pauses streams displayed on the Twitch frontpage",
		path: ["General", "Autopause"],
		defaultValue: false,
	}),
];
</script>
