<template lang="html">
  <div>
    <q-infinite-scroll :handler="loadMore" class="column wrap items-center" ref="list">
      <div class="row wrap justify-center">
        <div v-if="true" v-for="card in filteredCards" class="gallery-item">
          <card :avatar="card.image" :name="card.name"
            :number="card.number" :cardType="label" @openDetails="openDetails(card.number)"/>
        </div>
      </div>
      <q-spinner-dots slot="message" :size="40"></q-spinner-dots>
    </q-infinite-scroll>
    <q-fixed-position corner="bottom-right" :offset="[15, 20]">
      <q-btn class="animate-pop" color="primary" round
          v-back-to-top.animate="{offset: 1000, duration: 200}" >
        <q-icon name="keyboard_arrow_up" />
      </q-btn>
    </q-fixed-position>
  </div>
</template>

<script>
  import { QLayout, QInfiniteScroll, QSpinnerDots,
    QFixedPosition, QBtn, QIcon, BackToTop } from 'quasar'
  import Card from './Card.vue'
  import { getCards } from '../../js/fetch/index'

  export default {
    name: 'list',
    components: {
      QLayout,
      QInfiniteScroll,
      QSpinnerDots,
      QFixedPosition,
      QBtn,
      QIcon,
      Card
    },
    directives: {
      BackToTop
    },
    props: {
      searchFor: { default: '' },
      label: {
        required: true,
        type: String
      }
    },
    data () {
      return {
        cards: [],
        nextUrl: null
      }
    },
    computed: {
      filteredCards () {
        if (this.searchFor) {
          let filter = new RegExp(this.searchFor, 'i')
          return this.cards.filter(card => filter.test(card.name))
        }
        else {
          return this.cards
        }
      }
    },
    methods: {
      async loadMore (index, done) {
        let list = this.$refs.list

        if (this.cards.length === 0 || this.nextUrl) {
          let result = await getCards(this.nextUrl, this.label)
          this.nextUrl = result.nextUrl
          this.cards = this.cards.concat(result.cards)

          done()
          list.loadMore()
        }
        else {
          done()
          list.stop()
        }
      },
      openDetails (number) { this.$emit('openDetails', number) }
    }
  }
</script>

<style lang="css" scoped>
  .gallery-item {
    width: 10em;
  }
</style>
