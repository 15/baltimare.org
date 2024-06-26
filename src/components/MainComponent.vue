<script setup>
import { computed, onMounted, ref, watchEffect } from "vue";
import { useStore } from "@nanostores/vue";
import {
  updateData,
  avatarPosBaltimare,
  avatarPosHorseHeights,
  avatarPosObjBaltimare,
  avatarPosObjHorseHeights,
} from "../stores/store.js";
import parcels from "../data/parcels.js";

import { Tippy } from "vue-tippy";
import "tippy.js/dist/tippy.css";

const show = ref(true);

const $avatarPosBaltimare = useStore(avatarPosBaltimare);
const $avatarPosHorseHeights = useStore(avatarPosHorseHeights);
const $avatarPosObjBaltimare = useStore(avatarPosObjBaltimare);
const $avatarPosObjHorseHeights = useStore(avatarPosObjHorseHeights);

const $avatarPos = computed(() => {
  const baltimare = $avatarPosBaltimare.value.map((x) => ({
    ...x,
    location: "Baltimare",
  }));
  const horseHeights = $avatarPosHorseHeights.value.map((x) => ({
    ...x,
    location: "Horse Heights",
  }));
  return baltimare
    .concat(horseHeights)
    .filter(
      (x) =>
        !["BaltiMare Resident", "HorseHeights Resident"].includes(x.user_name)
    );
});

const sortedAvatarPos = computed(() => {
  return $avatarPos.value
    .sort((x, y) =>
      x.display_name.toUpperCase() < y.display_name.toUpperCase()
        ? -1
        : x.display_name.toUpperCase() > y.display_name.toUpperCase()
        ? 1
        : 0
    )
    .filter(
      (u) => u.display_name !== "BaltiMare" && u.display_name !== "Builder Pony"
    );
});

// const $avatarPosObj = computed(() => ({
//   ...$avatarPosObjBaltimare.value,
//   ...$avatarPosObjHorseHeights.value,
// }));

updateData();

onMounted(() => {
  updateData();

  dragElement(document.getElementById("mydiv"));
});

const magnify = (user) => {
  const pfp = document.getElementById(user.UUID);
  pfp.style.transition =
    "transform .3s ease-in-out, filter .3s ease-in-out, left 2s linear, bottom 2s linear, top 2s linear, right 2s linear";
  pfp.style.transform = "scale(1.75)";
  pfp.style.zIndex = 100000;
  pfp.style.filter = "drop-shadow(0 0 10px #ffcc00)";
  pfp.style.borderColor = "#ffcc00";
};

const demagnify = (user) => {
  const pfp = document.getElementById(user.UUID);
  pfp.style.transition =
    "transform .3s, filter .3s ease-in-out, left 2s linear, bottom 2s linear, top 2s linear 2s linear";
  pfp.style.transform = "scale(1)";
  pfp.style.zIndex = 10;
  pfp.style.filter = "";
  pfp.style.borderColor = "#ffffff";
};

const openDialog = (name) => {
  const dialog = document.getElementById(`dialog-${name}`);
  if (dialog instanceof HTMLDialogElement) {
    dialog.showModal();
    dialog.addEventListener("click", (e) => {
      if (e.target === dialog) {
        dialog.close();
      }
    });
  }
};

const teleportToUser = (user) => {
  const URL = `secondlife://${
    user.location === "Baltimare" ? "BALTIMARE" : "HORSE HEIGHTS"
  }/${user.user_position[0] || 0}/${user.user_position[1] || 0}/${
    user.user_position[2] || 0
  }`;
  window.location.href = URL;
};

function dragElement(elmnt) {
  console.log(elmnt);
  var pos1 = 0,
    pos2 = 0,
    pos3 = 0,
    pos4 = 0;
  if (document.getElementById(elmnt.id + "header")) {
    // if present, the header is where you move the DIV from:
    document.getElementById(elmnt.id + "header").onmousedown = dragMouseDown;
  } else {
    // otherwise, move the DIV from anywhere inside the DIV:
    elmnt.onmousedown = dragMouseDown;
  }

  function dragMouseDown(e) {
    e = e || window.event;
    e.preventDefault();
    // get the mouse cursor position at startup:
    pos3 = e.clientX;
    pos4 = e.clientY;
    document.onmouseup = closeDragElement;
    // call a function whenever the cursor moves:
    document.onmousemove = elementDrag;
  }

  function elementDrag(e) {
    e = e || window.event;
    e.preventDefault();
    // calculate the new cursor position:
    pos1 = pos3 - e.clientX;
    pos2 = pos4 - e.clientY;
    pos3 = e.clientX;
    pos4 = e.clientY;
    // set the element's new position:
    elmnt.style.top = elmnt.offsetTop - pos2 + "px";
    elmnt.style.left = elmnt.offsetLeft - pos1 + "px";
  }

  function closeDragElement() {
    // stop moving when mouse button is released:
    document.onmouseup = null;
    document.onmousemove = null;
  }
}
</script>

<template>
  <div
    style="filter: drop-shadow(0 2px 2px #000)"
    class="relative mx-4 w-full flex flex-col items-center justify-center rounded-lg"
  >
    <a
      target="_blank"
      href="https://boards.4chan.org/mlp/thread/41056119"
      class="w-full flex flex-col items-center justify-center cursor-pointer hover:opacity-40 duration-300"
    >
      <div
        class="w-64 lg:w-auto text-blue-300 text-6xl uppercase font-semibold tracking-wide"
      >
        <img
          src="/baltimare.webp"
          style="filter: drop-shadow(0 4px 4px #000)"
          width="700"
        />
      </div>
      <div
        class="w-32 lg:w-auto -mt-12 lg:-mt-32 text-blue-300 text-6xl uppercase font-semibold tracking-wide"
      >
        <img
          src="/opg.webp"
          style="filter: drop-shadow(0 4px 4px #000)"
          width="300"
        />
      </div>
    </a>
    <div class="w-full lg:w-5/6 relative flex rounded-lg mt-4">
      <img
        src="/mapcropped3.webp"
        class="w-full rounded-lg"
        usemap="#image-map"
        id="map"
      />
      <div
        v-for="(user, idx) in $avatarPos"
        :style="
          user.location === 'Baltimare'
            ? {
                left:
                  50 +
                  ($avatarPosObjBaltimare[user.UUID].user_position[0] / 256) *
                    50 -
                  0.5 +
                  '%',
                bottom:
                  ($avatarPosObjBaltimare[user.UUID].user_position[1] / 256) *
                    100 -
                  3 +
                  '%',
              }
            : {
                left:
                  ($avatarPosObjHorseHeights[user.UUID].user_position[0] /
                    256) *
                    50 -
                  0.5 +
                  '%',
                bottom:
                  ($avatarPosObjHorseHeights[user.UUID].user_position[1] /
                    256) *
                    100 -
                  3 +
                  '%',
              }
        "
        class="absolute flex flex-col items-center justify-center duration-[1500ms] z-50"
        :id="user.UUID"
        :key="user.UUID"
        @mouseover="magnify(user)"
        @mouseout="demagnify(user)"
      >
        <Tippy
          :content="`<div class='text-center'><h1 class='text-base font-medium'>${user.display_name.replace(
            ' Resident',
            ''
          )}</h1><h6 class='text-xs'>${user.user_name.replace(
            ' Resident',
            ''
          )}</h6></div>`"
          :allowHTML="true"
        >
          <img
            :src="`https://my-secondlife-agni.akamaized.net/users/${user.user_name
              .replace(' Resident', '')
              .replace(' ', '.')
              .toLowerCase()}/thumb_sl_image.png`"
            onerror="this.src='/twi.webp'"
            class="cursor-pointer glow bg-white min-w-6 max-w-6 w-6 min-h-6 max-h-6 h-6 rounded-full border border-white shadow-2xl z-50 duration-[1500ms]"
          />
        </Tippy>
      </div>
      <div
        v-for="parcel in parcels"
        :class="`cursor-pointer bg-white opacity-50 hover:opacity-90 border border-gray-400 absolute duration-300`"
        :style="
          parcel.location === 'Baltimare'
            ? {
                left: 50 + (parcel.coords[0][0] * 50) / 256 + '%',
                bottom: (parcel.coords[0][1] * 100) / 256 + '%',
                width:
                  ((parcel.coords[3][0] - parcel.coords[0][0]) * 50) / 256 +
                  '%',
                height:
                  ((parcel.coords[1][1] - parcel.coords[0][1]) * 100) / 256 +
                  '%',
              }
            : {
                left: (parcel.coords[0][0] * 50) / 256 + '%',
                bottom: (parcel.coords[0][1] * 100) / 256 + '%',
                width:
                  ((parcel.coords[3][0] - parcel.coords[0][0]) * 50) / 256 +
                  '%',
                height:
                  ((parcel.coords[1][1] - parcel.coords[0][1]) * 100) / 256 +
                  '%',
              }
        "
        @click="openDialog(parcel.name)"
      >
        <Tippy
          :content="`<div class='text-center'><h1 class='text-base font-medium'>${parcel.name}</h1><h6 class='text-xs'>Owned by <span class='font-semibold'>${parcel.owner}</span></h6></div>`"
          :allowHTML="true"
          :placement="'right'"
        >
          <div
            :class="`${
              parcel.rotate === 45
                ? '-rotate-45'
                : parcel.rotate === 90
                ? '-rotate-90'
                : 'rotate-0'
            } w-full h-full text-[0.8rem] font-semibold text-black  opacity-100 flex items-center justify-center whitespace-nowrap`"
          >
            <!-- {{
              parcel.coords[3][0] - parcel.coords[0][0] > 40
                ? parcel.name
                : null
            }} -->
            {{ parcel.label ?? null }}
          </div>
        </Tippy>
      </div>
      <dialog
        :style="{ 'font-family': 'Poppins' }"
        class="p-0 cursor-default max-h-screen overflow-hidden"
        v-for="parcel in parcels"
        :id="`dialog-${parcel.name}`"
      >
        <div
          class="block md:hidden absolute left-2 top-2 hover:bg-red-200 rounded-lg text-stone-600"
          id="close-m"
        >
          <svg
            width="40"
            height="40"
            viewBox="0 0 24 24"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M6.2253 4.81108C5.83477 4.42056 5.20161 4.42056 4.81108 4.81108C4.42056 5.20161 4.42056 5.83477 4.81108 6.2253L10.5858 12L4.81114 17.7747C4.42062 18.1652 4.42062 18.7984 4.81114 19.1889C5.20167 19.5794 5.83483 19.5794 6.22535 19.1889L12 13.4142L17.7747 19.1889C18.1652 19.5794 18.7984 19.5794 19.1889 19.1889C19.5794 18.7984 19.5794 18.1652 19.1889 17.7747L13.4142 12L19.189 6.2253C19.5795 5.83477 19.5795 5.20161 19.189 4.81108C18.7985 4.42056 18.1653 4.42056 17.7748 4.81108L12 10.5858L6.2253 4.81108Z"
              fill="currentColor"
            />
          </svg>
        </div>
        <div
          class="overflow-hidden p-4 md:p-8 bg-blue-50 flex flex-col items-center justify-center w-full h-auto sm:w-120 md:w-160 lg:w-240 xl:w-288"
        >
          <div
            class="flex items-center justify-center w-1/2 my-8"
            v-if="parcel.img.length === 1"
          >
            <img
              loading="lazy"
              :src="
                parcel.img ? `${parcel.img[0]}` : '/Feast_Hall_Preparations.png'
              "
              height="800"
              style="filter: drop-shadow(0 2px 2px #000)"
              class="w-auto z-0 h-48 sm:h-64 lg:h-auto lg:min-w-48 bg-white rounded-lg -rotate-3 p-4"
            />
          </div>
          <div
            class="min-h-96 my-4 relative flex flex-col items-center justify-center w-full"
            v-else-if="parcel.img.length === 2"
          >
            <img
              loading="lazy"
              :src="
                parcel.img[0]
                  ? `${parcel.img[0]}`
                  : '/Feast_Hall_Preparations.png'
              "
              height="800"
              style="filter: drop-shadow(0 2px 2px #000)"
              class="z-0 mr-20 -mb-16 w-1/2 h-48 sm:h-64 lg:h-auto lg:min-w-48 bg-white rounded-lg -rotate-3 p-3"
            />
            <img
              loading="lazy"
              :src="
                parcel.img[1]
                  ? `${parcel.img[1]}`
                  : '/Feast_Hall_Preparations.png'
              "
              height="800"
              style="filter: drop-shadow(0 2px 2px #000)"
              class="z-10 ml-20 -mt-16 w-1/2 h-48 sm:h-64 lg:h-auto lg:min-w-48 bg-white rounded-lg rotate-3 p-3"
            />
          </div>
          <div
            class="min-h-96 mt-4 relative flex items-center justify-center w-full mb-16"
            v-else-if="parcel.img.length === 3"
          >
            <img
              loading="lazy"
              :src="
                parcel.img[0]
                  ? `${parcel.img[0]}`
                  : '/Feast_Hall_Preparations.png'
              "
              height="800"
              style="filter: drop-shadow(0 2px 2px #000)"
              class="top-4 left-8 absolute z-0 w-1/2 h-48 sm:h-64 lg:h-auto lg:min-w-48 bg-white rounded-lg -rotate-3 p-3"
            />
            <img
              loading="lazy"
              :src="
                parcel.img[1]
                  ? `${parcel.img[1]}`
                  : '/Feast_Hall_Preparations.png'
              "
              height="800"
              style="filter: drop-shadow(0 2px 2px #000)"
              class="-bottom-8 absolute z-10 w-1/2 h-48 sm:h-64 lg:h-auto lg:min-w-48 bg-white rounded-lg rotate-3 p-3"
            />
            <img
              loading="lazy"
              :src="
                parcel.img[2]
                  ? `${parcel.img[2]}`
                  : '/Feast_Hall_Preparations.png'
              "
              height="800"
              style="filter: drop-shadow(0 2px 2px #000)"
              class="top-4 right-8 absolute z-0 w-1/2 h-48 sm:h-64 lg:h-auto lg:min-w-48 bg-white rounded-lg -rotate-1 p-3"
            />
          </div>
          <div class="flex flex-col gap-y-2 w-auto lg:ml-12">
            <div
              class="text-5xl lg:text-6xl xl:text-7xl text-center tracking-wide font-semibold uppercase"
            >
              {{ parcel.name }}
            </div>
            <div
              class="text-lg lg:text-xl xl:text-2xl text-center tracking-wide font-medium uppercase"
            >
              Owned by <span class="text-neutral-500">{{ parcel.owner }}</span>
            </div>
            <a
              style="filter: drop-shadow(0 1px 1px #000)"
              :href="`secondlife://${parcel.location.toUpperCase()}/${
                parcel?.teleport?.[0] || 0
              }/${parcel?.teleport?.[1] || 0}/${parcel?.teleport?.[2] || 0}`"
              :class="`text-xl tracking-wide rounded bg-blue-400 hover:bg-blue-600 duration-200 text-center mx-auto px-3 py-1 uppercase text-white cursor-pointer`"
            >
              Teleport here
            </a>
          </div>
          <div class="mt-4 flex flex-col w-full xl:w-full lg:ml-12">
            <div
              class="md:py-0 text-xl lg:text-2xl xl:text-3xl text-center tracking-wide font-semibold uppercase"
            >
              Current occupants:
              {{
                sortedAvatarPos.filter(
                  (u) =>
                    u.location === parcel.location &&
                    u.user_position[0] >= parcel.coords[0][0] &&
                    u.user_position[0] <= parcel.coords[3][0] &&
                    u.user_position[1] >= parcel.coords[0][1] &&
                    u.user_position[1] <= parcel.coords[1][1]
                ).length
              }}
            </div>
            <div class="mt-4 grid grid-cols-5 gap-x-6 gap-y-0">
              <div
                class="flex gap-x-1 items-center whitespace-nowrap"
                v-for="user in sortedAvatarPos.filter(
                  (u) =>
                    u.location === parcel.location &&
                    u.user_position[0] >= parcel.coords[0][0] &&
                    u.user_position[0] <= parcel.coords[3][0] &&
                    u.user_position[1] >= parcel.coords[0][1] &&
                    u.user_position[1] <= parcel.coords[1][1]
                )"
              >
                <img
                  :src="user.pfp"
                  class="w-4 h-4 rounded"
                  onerror="this.src='/twi.webp'"
                />
                <span class="text-sm">{{ user.display_name }}</span>
              </div>
            </div>
          </div>
        </div>
      </dialog>
    </div>
    <div
      id="mydiv"
      :class="`z-50 absolute bg-amber-50 opacity-90 hover:bg-amber-100 active:opacity-50 cursor-move rotate-0 lg:rotate-3 lg:translate-x-[135%] px-4 py-4 rounded-2xl`"
      style="
        filter: drop-shadow(0 2px 2px #000);
        transition-duration: 300ms;
        transition-property: background-color, opacity;
      "
    >
      <div
        class="text-2xl text-black font-semibold text-center"
        id="myDivheader"
      >
        <span class="font-semibold uppercase">Online: </span>
        <span class="font-semibold">{{
          $avatarPos.length
            ? Math.max(
                0,
                $avatarPos.filter((x) => x.location === "Baltimare").length - 1
              ) +
              Math.max(
                0,
                $avatarPos.filter((x) => x.location === "Horse Heights").length
              )
            : "?"
        }}</span>
        / 220
      </div>
      <div class="my-1 flex flex-col items-center">
        <div class="lg:text-sm text-black font-normal text-center">
          <span class="font-medium uppercase">Baltimare: </span>
          <span class="font-normal">{{
            Math.max(
              0,
              $avatarPos.filter((x) => x.location === "Baltimare").length - 1
            )
          }}</span>
          / 110
        </div>
        <div class="text-sm text-black font-normal text-center">
          <span class="font-medium uppercase">Horse Heights: </span>
          <span class="font-normal">{{
            Math.max(
              0,
              $avatarPos.filter((x) => x.location === "Horse Heights").length
            )
          }}</span>
          / 110
        </div>
      </div>
      <div
        v-if="!$avatarPos.length"
        class="min-w-120 max-w-120 w-120 columns-1"
      >
        <img class="max-w-120 w-120" src="/loading.webp" />
        <div class="text-center mt-1 text-lg">Loading users...</div>
      </div>
      <div v-else-if="$avatarPos.length > 0">
        <div class="min-w-120 w-120 columns-2">
          <div
            v-for="(user, idx) in sortedAvatarPos"
            @mouseover="magnify(user)"
            @mouseout="demagnify(user)"
            class="hidden lg:block w-full"
          >
            <div
              class="w-auto whitespace-nowrap rounded hover:bg-neutral-800 text-black hover:text-amber-50 px-1 duration-300 cursor-move font-medium gap-x-2 flex items-center justify-start"
            >
              <img
                :src="user.pfp"
                class="w-4 h-4 rounded"
                onerror="this.src='/twi.png'"
              />
              <span class="text-xs lg:text-sm font-medium">
                {{ user.display_name.replace(" Resident", "") }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
