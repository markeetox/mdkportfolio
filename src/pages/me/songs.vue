<script lang="ts">
import Vue from "vue"

// Netlify function return type
interface LastFmUser {
  name: string
  registered: number
  totalPlays: number
  image: string
  url: string
}

interface LastFmSong {
  name: string
  artist: string
  image: string
  url: string
  date: number
  plays?: number
  nowPlaying: boolean
}

interface LastFmArtist {
  name: string
  plays: number
  image: string
  url: string
}

interface LastFmResponse {
  user: LastFmUser
  topTracks: LastFmSong[]
  recentTracks: LastFmSong[]
  topArtists: LastFmArtist[]
}

export default Vue.extend({
  data() {
    return {
      lastFm: {} as LastFmResponse,
    }
  },
  fetchOnServer: false,
  async fetch() {
    const url =
      process.env.NODE_ENV === "production"
        ? "https://eggsy.xyz/.netlify/functions/getLastFmSongs"
        : "http://localhost:8888/.netlify/functions/getLastFmSongs"

    const { data: songs }: { data: LastFmResponse } = await this.$axios(url)

    this.lastFm = songs
  },
  head() {
    const title = "My Songs"
    const description =
      "Songs that I recently listened and the songs that I listened most as well as some more information from Last.fm, all of that information is on this page!"

    return {
      title,
      meta: this.$prepareMeta({
        title,
        description,
      }),
    }
  },
})
</script>

<template>
  <div class="text-gray-500 dark:text-neutral-600">
    <header class="space-y-2 my-12 px-4">
      <h1 class="text-gray-900 text-4xl dark:text-neutral-300">Songs</h1>
      <p>
        My latest activity on Last FM. Don't forget to check out my
        <SmartLink href="/daily" class="underline"
          >daily song suggestions</SmartLink
        >
        page as well!
      </p>
    </header>

    <LoadersSongs
      v-if="$fetchState.pending === true || $fetchState.error !== null"
    />

    <main v-else class="space-y-20">
      <div>
        <Title class="mb-4">Details</Title>

        <div class="grid px-4 gap-x-0 gap-y-4 md:gap-x-12 md:grid-cols-2">
          <!-- Profile -->
          <div class="flex space-x-4 items-center justify-between">
            <span>Profile</span>

            <div class="flex space-x-2 items-center">
              <SmartLink
                href="https://last.fm/user/eggsywashere"
                class="flex-shrink-0"
                blank
                >@{{ lastFm.user.name }}</SmartLink
              >

              <SmartImage
                :src="lastFm.user.image"
                class="rounded-full h-6 w-6"
              />
            </div>
          </div>

          <!-- Play count -->
          <div class="flex space-x-4 items-center justify-between">
            <span class="flex-shrink-0">Total Plays</span>

            <div class="flex space-x-2 items-center">
              <div class="truncate">{{ lastFm.user.totalPlays }}</div>
              <IconFire filled class="h-6 text-red-700 w-6 dark:text-current" />
            </div>
          </div>

          <!-- Registered -->
          <div class="flex space-x-4 items-center justify-between">
            <span class="flex-shrink-0">Account Age</span>

            <div class="flex space-x-2 items-center">
              <div class="truncate">
                {{
                  new Date().getFullYear() -
                  new Date(lastFm.user.registered * 1000).getFullYear()
                }}

                year(s)
              </div>

              <IconCalendar class="h-6 w-6" />
            </div>
          </div>
        </div>
      </div>

      <div>
        <Title class="mb-4">Top Songs (last 7 days)</Title>

        <div class="grid gap-2 md:grid-cols-2">
          <CardLastFm
            v-for="song of lastFm.topTracks"
            :key="song.name"
            :name="song.name"
            :artist="song.artist"
            :image="song.image"
            :now-playing="song.nowPlaying"
            :plays="song.plays"
            :url="song.url"
          />
        </div>
      </div>

      <div>
        <Title class="mb-4">Top Artists (last 7 days)</Title>

        <div class="grid gap-2 md:grid-cols-2">
          <CardLastFm
            v-for="artist of lastFm.topArtists"
            :key="artist.name"
            :name="artist.name"
            :plays="artist.plays"
            :image="artist.image"
            :url="artist.url"
          />
        </div>
      </div>

      <div>
        <Title class="mb-4">Recent Songs</Title>

        <div class="grid gap-2 md:grid-cols-2">
          <CardLastFm
            v-for="song of lastFm.recentTracks"
            :key="song.name"
            :name="song.name"
            :artist="song.artist"
            :image="song.image"
            :now-playing="song.nowPlaying"
            :url="song.url"
          />
        </div>
      </div>
    </main>
  </div>
</template>
