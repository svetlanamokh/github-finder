<template>
  <div class="wrapper-content wrapper-content--fixed">
    <section>
      <div class="container">

        <!-- erorrs -->
        <div class="error" v-if="error" style="margin-bottom: 20px;">
          <p> {{ error }} </p>
        </div>

        <!-- search -->
        <search
          :value="search"
          placeholder="Type username..."
          @search="search = $event"/>

        <!-- buttons -->
        <button v-if="!repos" class="btn btnPrimary" @click="getData()">Search!</button>
        <button v-else class="btn btnPrimary" @click="getData()">Search Again!</button>

        <!-- wrapper -->
        <div class="repos__wrapper" v-if="repos">
          <div class="repos-sort">
            <div @click="sort('name')">Name &#8595;</div>
            <div @click="sort('stargazers_count')">Stars &#8595;</div>
          </div>
          <!-- item -->
          <div class="repos-item" v-for="repo in reposSort" :key="repo.id">
            <div class="repos-info">
              <a class="link" target="_blank" :href="repo.html_url">{{ repo.name }}</a>
              <span> {{ repo.stargazers_count }} ⭐</span>
            </div>
          </div>
        </div>
        <div class="users__wrapper" v-if="users">
          <div class="users-info">
            <img class="user-photo" :src="users.avatar_url" alt="photo">
            <a class="link" target="_blank" :href="users.html_url">{{ users.login }}</a>
            <div>{{ users.public_repos }}</div>
          </div>
        </div>
      </div>     
    </section>

    <section>
      <div class="container">
          <div class="button-list">
              <div class="btn btnPrimary" v-if="repos" @click="prevPage"> &#8592; </div>
              <div class="btn btnPrimary" v-if="repos" @click="nextPage"> &#8594; </div>
          </div>
      </div>
    </section>

  </div>
</template>

<script>
import search from '@/components/Search.vue'
import axios from 'axios'
export default {
  components: { search },
  data () {
    return {
      search: '',
      error: null,
      repos: null,
      users: null,
      currentSort: 'name', 
      currentSortDir: 'asc',
      page: {
        current: 1,
        length: 3
      }
    }
  },
  computed: {
    reposSort () {
      return this.repos.sort((a, b) => {
          let mod = 1
          if (this.currentSortDir === 'desc') mod = -1
          if (a[this.currentSort] < b[this.currentSort]) return -1 * mod
          if (a[this.currentSort] > b[this.currentSort]) return 1 * mod
          return 0
      }).filter((row, index) => {
          let start = (this.page.current-1) * this.page.length
          let end = this.page.current * this.page.length
          if (index >= start && index < end) return true
      })
    }
  },
  methods: {
    getRepos () {
      return axios.get(`https://api.github.com/users/${this.search}/repos`)
    },
    getUsers () {
      return axios.get(`https://api.github.com/users/${this.search}`)
    },
    getData () {
      Promise.all([this.getRepos(), this.getUsers()])
        .then((res) => {
          this.error = null
          this.repos = res[0].data
          this.users = res[1].data
          console.log(res)
        })
        .catch(err => {
          console.log(err)
          this.repos = null
          this.error = 'Can`t find this user'
          this.users = ''
        })
      
    },
    sort (e) {
      if (e === this.currentSort) {
        this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc'
      }
      this.currentSort = e
    },
    prevPage () {
      if (this.page.current > 1) this.page.current-=1         
    },
    nextPage () {
      if ((this.page.current * this.page.length) < this.repos.length) this.page.current+=1
    }

  }
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  align-items: center;
  flex-direction: column;
}
button {
  margin-top: 40px;
}
.repos__wrapper {
  width: 400px;
  margin: 30px 0 0 0;
}

.repos-sort {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.repos-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  margin-bottom: 10px;
  border-bottom: 1px solid #dbdbdb;
}

.users__wrapper {
  width: 400px;
}

.users-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #dbdbdb;
}

.user-photo {
  width: 50px;
  height: 50px;
}

.button-list {
  text-align: center;
  width: 100%;
}

.btn {
  border-radius: 60px;
}

</style>
