<template>
  <section :class="{ queries: true, loading }">
    <header v-if="queries && queries.length">
      <div class="query-name">
        <v-select
          placeholder="Choose a Query"
          :value="queryName"
          :options="queries"
          :on-change="setQuery"
          :searchable="false"
        />
      </div>
      <a :href="queryName && !running ? '#' : undefined" :class="{ run: true, running }" @click.prevent="run">Run</a>
    </header>
    <pre v-if="result">{{result}}</pre>
    <span class="error" v-if="error">{{error}}</span>
    <span class="no-queries" v-if="queries && queries.length === 0">No queries registered</span>
  </section>
</template>

<script>
import moment from 'moment'

export default {
  data() {
    return {
      loading: false,
      queryName: undefined,
      queryInput: undefined,
      queries: undefined,
      result: undefined,
      error: undefined,
      running: false
    }
  },
  created() {
    this.loading = true
    this.$http(this.$parent.baseAPIURL + '/queries').then(
      r => {
        this.queries = r.filter(r => r !== '__stack_trace')
        if (!this.queryName) {
          this.queryName = this.queries[0]
        }
      },
      e => this.error = (e.json && e.json.message) || e.status || e.message
    ).finally(() => this.loading = false)
  },
  methods: {
    setQuery(queryName) {
      this.result = undefined
      this.error = undefined
      this.queryName = queryName
    },
    run() {
      this.running = true
      this.$http.post(`${this.$parent.baseAPIURL}/queries/${this.queryName}`)
        .then(r => this.result = r.queryResult, e => this.error = (e.json && e.json.message) || e.status || e.message)
        .finally(() => this.running = false)
    }
  }
}
</script>

<style lang="stylus">
@require "../../styles/definitions.styl"

section.queries
  padding layout-spacing-small
  header
    display flex
    padding-top layout-spacing-small
    margin-bottom layout-spacing-medium
    align-items center
    .query-name
      flex 0 0 auto
      min-width 350px
      superlabel()
      &::before
        top -16px
        content 'query'
    a.run
      flex 0 0 auto
      margin 0 1em
      action-button()
      icon-play()
      &:not([href="#"])
        opacity 0.7
      &.running
        icon-refresh()

  span.no-queries
    display block
    width 100%
    text-align center
    font-size 16px
    color uber-black-60
</style>