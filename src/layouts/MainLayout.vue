<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class = "no-print">
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          <input id="s" type="search" name="s" v-model="myKey" list="words" :placeholder="s('輸入字詞')" @keydown.enter="$router.push('/w/' + myKey)"/>
          <label for="s"></label>
          <datalist id ="words">
            <option v-for = "d in has(data, myKey).slice(0,n)" :key="d" :value="d"></option>
            }
          </datalist>
          <button @click="$router.push('/w/' + myKey)">{{ s('查詢') }}</button>
        </q-toolbar-title>
        <a href="https://www.github.com/bestian/q-moedict/" target="_blank">
          <img class = "icon" src="../assets/github-icon.svg" title="Fork Me On Github" width="16" height="16" />
        </a>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-1"
      class = "no-print"
    >
    <q-list bordered>
      <q-item clickable to = "/w/萌">{{ s('國語萌典') }}</q-item>
      <q-item clickable to = "/w/~萌">{{ s('兩岸萌典') }}</q-item>
      <q-item clickable to = "/w/'發穎">{{ s('臺灣閩南語') }}</q-item>
      <q-item clickable to = "/w/:發芽">{{ s('臺灣客家語') }}</q-item>
      <q-item>
        <b>{{ s('已加星號') }}</b>
      </q-item>
      <q-item clickable v-for = "k in stars" :to = "'/w/' + k" :key="k">
        {{s(k)}}
      </q-item>
    </q-list>
    </q-drawer>

    <q-page-container>
      <div class="gcse-search"></div>
      <router-view @updateStars = "updateStars()" @updateSi = "s1" :stars="stars"/>
    </q-page-container>
  </q-layout>
</template>

<script>
import { sify } from 'chinese-conv'

export default {
  name: 'MainLayout',
  data () {
    return {
      myKey: '',
      data: [],
      stars: [],
      leftDrawerOpen: false,
      n: 100,
      si: false
    }
  },
  methods: {
    s (t) {
      if (this.si) {
        return sify(t)
      } else {
        return t
      }
    },
    s1 () {
      console.log('S1')
      var si = this.$q.localStorage.getItem('si')
      console.log(si)
      if (!si) { si = false }
      this.si = si
      this.$q.localStorage.set('si', si)
    },
    deep () {
      var p = this.s('全站搜尋')
      window.IS_GOOGLE_AFS_IFRAME_ = true
      const cx = '007966820757635393756:sasf0rnevk4'
      var gcse = document.createElement('script')
      gcse.type = 'text/javascript'
      gcse.async = true
      gcse.src = '//www.google.com/cse/cse.js?cx=' + cx
      var s = document.getElementsByTagName('script')[0]
      s.parentNode.insertBefore(gcse, s)
      setInterval(function () {
        var e = document.getElementById('gsc-i-id1')
        if (e) {
          e.setAttribute('placeholder', p)
        }
      }, 500)
    },
    onLoad () {
      this.n += 50
    },
    has (data, k) {
      // console.log(data)
      if (!k) {
        return []
      }
      return data.filter((x) => { return x.indexOf(k) > -1 })
    },
    updateStars () {
      this.stars = this.$q.localStorage.getItem('words') || []
    }
  },
  mounted () {
    this.s1()
    var vm = this
    this.deep()
    this.$axios.get('https://www.moedict.tw/a/index.json')
      .then((response) => {
        this.data = response.data
      })
    this.stars = this.$q.localStorage.getItem('words') || []
    setInterval(function () {
      var list = document.getElementsByClassName('gs-title')
      for (var i = 0; i < list.length; i++) {
        const e = list[i]
        if (e.getAttribute('href')) {
          var l = e.getAttribute('href')
          l = ('' + l).replace('https://www.moedict.tw/', '')
          e.removeAttribute('href')
          e.removeAttribute('data-cturl')
          e.removeAttribute('data-ctorig')
          e.setAttribute('data-h', l)
          e.setAttribute('target', 'self')
          e.setAttribute('rel', 'noreferrer')
          e.addEventListener('click', function (event) {
            event.preventDefault()
            console.log(this.getAttribute('data-h'))
            vm.$router.push('/w/' + this.getAttribute('data-h'))
            var x = document.getElementsByClassName('gsc-modal-background-image gsc-modal-background-image-visible')
            x[0].click()
            vm.$forceUpdate()
          })
        }
      }
    }, 1000)
  }
}
</script>

<style type="text/css">
  #s {
    width: 130px !important;
  }

  @media screen and (min-width: 420px) {
    #s {
      width: 250px !important;
    }
  }

  .icon {
    height: 1.2em;
  }

  .ellipsis {
    text-overflow: inherit;
  }
</style>
