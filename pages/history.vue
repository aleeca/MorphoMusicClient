<template>
  <v-container>
    <template v-if="!liffErr">
      <v-row justify="center" align="center" class="mb-3">
        <v-spacer> </v-spacer>
        <v-col cols="8" class="text-center">
          <h1>Morphomusic</h1>
        </v-col>
        <v-col class="text-right" cols="2">
          <v-avatar v-if="pictureUrl" size="36">
            <img :src="pictureUrl" alt="プロフィール画像" />
          </v-avatar>
        </v-col>
      </v-row>
      <p v-if="isLoggedin" class="text-center">
        <strong>{{ displayName }}</strong
        >さんの履歴
      </p>
      <v-row v-if="songsData">
        <v-col v-for="item in songsData" :key="item.id" cols="12" sm="6" md="4">
          <v-card elevation="3" class="text-center">
            <v-avatar class="ma-3" size="200" tile>
              <v-img :src="item.artwork_url"></v-img>
            </v-avatar>
            <h2>{{ item.song_name }}</h2>
            <v-card-subtitle>{{ item.artist_name }}</v-card-subtitle>
            <v-btn class="mb-3 play-btn" @click="buyMusic(item.buy_url)">
              PLAY
              <v-icon right>mdi-spotify</v-icon>
            </v-btn>
          </v-card>
        </v-col>
      </v-row>
    </template>
    <template v-else>
      <v-row justify="center" align="center" class="mb-3">
        <v-col cols="12" class="text-center">
          <h1>{{ liffErrtext }}</h1>
        </v-col>
      </v-row>
    </template>
  </v-container>
</template>

<script>
let liff
if (process.client) {
  liff = require('@line/liff')
}

export default {
  data() {
    return {
      isinLine: false,
      isLoggedin: false,
      userId: '',
      displayName: '',
      pictureUrl: '',
      idToken: '',
      liffErr: false,
      liffErrtext: '',
      songsData: null,
    }
  },
  head: {
    title: 'Morphomusic',
  },
  created() {
    if (process.client) {
      if (liff.isInClient()) {
        this.isinLine = true
      } else {
        this.liffErr = true
        this.liffErrtext = 'LINEアプリ内で表示してください'
      }
      liff
        .init({
          liffId: process.env.LIFFID,
        })
        .then(() => {
          this.idToken = liff.getIDToken()
          this.initializeApp()
        })
        .catch(() => {
          this.liffErr = true
          this.liffErrtext = 'LIFFアプリのイニシャライズに失敗しました'
        })
    }
  },
  methods: {
    initializeApp() {
      if (liff.isLoggedIn()) {
        this.isLoggedin = true
        this.getProfile()
        this.getData()
      } else {
        liff.login()
      }
    },
    getProfile() {
      liff
        .getProfile()
        .then((profile) => {
          this.userId = profile.userId
          this.displayName = profile.displayName
          this.pictureUrl = profile.pictureUrl
        })
        .catch(() => {
          this.liffErr = true
          this.liffErrtext = 'ユーザー情報の取得に失敗しました'
        })
    },
    async getData() {
      const url = 'https://morphomusic.herokuapp.com/'
      const data = await this.$axios.$get(url, {
        headers: {
          Authorization: `Idtoken ${this.idToken}`,
        },
      })
      const obj = JSON.parse(JSON.stringify(data))
      if (obj.status === 'failed') {
        alert('情報が取得できませんでした')
      } else {
        this.songsData = obj.songs
      }
    },
    buyMusic(link) {
      liff.openWindow({
        url: link,
        external: true,
      })
    },
  },
}
</script>

<style scoped>
.v-btn.play-btn {
  background-color: #1db954 !important;
  color: #ffffff;
}
</style>
