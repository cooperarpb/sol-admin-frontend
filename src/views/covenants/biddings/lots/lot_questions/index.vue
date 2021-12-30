<style scoped lang="scss">
</style>

<template lang="pug">
  .root
    headers(:title="this.$t('.title')")
    .container.tool.biddings.u-cf(v-if="isLoading")
      .alert.alert-info
        | {{ $t('.loading') }}

    .container.tool.biddings.u-cf(v-else-if="this.lot_questions_count > 0")
      .card.mb-2.u-cf.u-full-width
        ul.mb-0
          li.list-item.row.p-1(v-for="lotQuestion in lot_questions")
            .container
              .list-title
                | {{ lotQuestion.question }}
              span(v-if="lotQuestion.answer")
                | {{ lotQuestion.answer }}
              span(v-else)
                | {{ $t('.waiting_answer') }}

    .container.tool.biddings.u-cf(v-else)
      .alert.alert-info
        | {{ $t('.empty') }}

    .row
      .twelve.columns
        paginator(
          v-model="page",
          :first="firstPageLink",
          :prev="prevPageLink",
          :next="nextPageLink",
          :last="lastPageLink",
          v-if="this.lot_questions_count"
        )

</template>

<script>
  import parseLinkHeaders from 'parse-link-header'

  export default {
    name: 'lotQuestions',
    data () {
      return {
        i18nScope: 'covenants.biddings.lots.lot_questions.index',

        // resource
        lot_questions: null,
        lot_questions_count: 0,
        params: {},
        isLoading: true,

        // pagination
        page:          1,
        firstPageLink: '',
        prevPageLink:  '',
        nextPageLink:  '',
        lastPageLink:  '',
        totalPages:    1,
      }
    },

    computed: {
      fetchParams() {
        return this.params
      }
    },

    methods: {
      fetch() {
        let params = this.params

        this.isLoading = true

        return this.$http.get('/administrator/covenants/' + this.covenantId + '/biddings/' + this.biddingId + '/lots/' + this.lotId + '/lot_questions')
          .then((response) => {

            this.isLoading = false

            this.lot_questions = response.data
            this.lot_questions_count = this.lot_questions.length

            this.updatePagination(response)

          }).catch((_err) => {
            this.error = _err
            console.error(_err)
          })
      },

      updatePagination(aResponse) {
        this.page = aResponse.headers['x-page']
        this.totalPages = aResponse.headers['x-total']
        let links = parseLinkHeaders(aResponse.headers.link) || {}

        this.firstPageLink = _.dig(links, 'first', 'page')
        this.prevPageLink = _.dig(links, 'prev', 'page')
        this.nextPageLink = _.dig(links, 'next', 'page')
        this.lastPageLink = _.dig(links, 'last', 'page')
      },

      parseRoute() {
        this.covenantId = this.$params.asInteger(this.$route.params.covenant_id)
        this.biddingId = this.$params.asInteger(this.$route.params.bidding_id)
        this.lotId = this.$params.asInteger(this.$route.params.lot_id)
      },

      updateRoute() {
        let data = Object.assign({}, this.$route.query, this.params);

        this.$router.replace({ name: 'LotQuestions', query: data });
      },

      init() {
        this.parseRoute()
        this.params = this.$route.query
      }
    },

    created: function () {
      this.init();
    },

    watch: {
      fetchParams() {
        this.fetch()
      },

      page() {
        this.params = Object.assign({}, this.params, { page: this.page });
        this.updateRoute()
      }
    }
  }

</script>
