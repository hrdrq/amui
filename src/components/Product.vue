<template>
  <div>
    <div class="row">
      <div class="col-2">
        <q-input v-model="title" placeholder="商品名" @keydown.enter="reset();search()" autofocus />
      </div>
      <div class="col-2">
        <q-input v-model="editorial" placeholder="説明" @keydown.enter="reset();search()" />
      </div>
      <div class="col-2">
        <q-input v-model="feature" placeholder="属性" @keydown.enter="reset();search()" />
      </div>
    </div>
    <q-infinite-scroll v-if="productList.length>0" :handler="refresher">
      <q-card v-for="(p, index) in productList" :key="index">
        <q-card-main>
          <div>
            <a target="_blank" :href="p.DetailPageURL">
              <div style="color:black">{{category(p.BrowseNodes)}}</div>
              <h5 v-html="highlight(p.ItemAttributes.Title, title)" />
              <div v-if="p.ImageSets">
                <template v-if="Array.isArray(p.ImageSets.ImageSet)">
                  <img v-for="i in p.ImageSets.ImageSet" :src="i.MediumImage.URL">
                </template>
                <template v-else>
                  <img :src="p.ImageSets.ImageSet.MediumImage.URL">
                </template>
              </div>
            </a>
          </div>
          <q-card-separator />
          <div class="row detail" @click="window.open(p.DetailPageURL,'_blank')">
            <div class="col-3">
              <!-- <div>
                <img :src="p.MediumImage.URL">
              </div> -->
              <div>
                <h4 v-if="p.OfferSummary&&p.OfferSummary.LowestNewPrice">{{p.OfferSummary.LowestNewPrice.FormattedPrice}}</h4>
              </div>
              <div v-if="p.rate_val">
                <q-rating
                  readonly
                  v-model="p.rate_val"
                  size="20px"
                  color="red"
                  :max="5" />
                <span>{{p.rate_num}}</span>
              </div>
              <div v-if="p.SalesRank" :style="{ 'font-size': rankSize(p.SalesRank), 'color': 'green' }">
                {{Number(p.SalesRank).toLocaleString()}}
              </div>
              <div>
                <div>Publisher：{{p.ItemAttributes.Publisher}}</div>
                <div>Manufacturer：{{p.ItemAttributes.Manufacturer}}</div>
                <div>Brand：{{p.ItemAttributes.Brand}}</div>
              </div>
            </div>
            <div class="col-9 intro">
              <div v-for="f in p.ItemAttributes.Feature">{{f}}</div>
              <hr>
              <div v-if="p.EditorialReviews">
                <div v-if="Array.isArray(p.EditorialReviews.EditorialReview)" v-for="e in p.EditorialReviews.EditorialReview">
                  <div v-html="e.Content" />
                </div>
                <div v-else  v-html="p.EditorialReviews.EditorialReview.Content" />
              </div>
            </div>
          </div>
          <q-card-separator />
          <div class="row">
            <div class="col">
              <q-btn big class="full-width" @click="dislike(p._id)">イマイチ</q-btn>
            </div>
            <div class="col">
              <q-btn big class="full-width" @click="favorite(p._id, index)">
                <span v-if="p.favorite" style="color:red"><q-icon name="star" /></span>
                <span v-else><q-icon name="star border" /></span>
                お気に入り
              </q-btn>
            </div>
          </div>
        </q-card-main>
      </q-card>
      <div class="row justify-center">
        <q-spinner-dots slot="message" :size="40" />
      </div>
    </q-infinite-scroll>
  </div>
</template>

<script>
import {
  Alert,
  Toast,
  QCard,
  QCardTitle,
  QCardMain,
  QCardActions,
  QCardMedia,
  QCardSeparator,
  QBtn,
  QIcon,
  QModal,
  QTooltip,
  QInfiniteScroll,
  QSpinnerDots,
  QInput,
  QRating
} from 'quasar'
import axios from 'axios'
const LIMIT = 50
const HOST = '//localhost:8891'
export default {
  name: 'user',
  data () {
    return {
      window: window,
      title: '台湾製',
      editorial: null,
      feature: null,
      productList: [],
      offset: 0
    }
  },
  methods: {
    rankSize (rank) {
      if (rank > 1000000) {
        return '100%'
      }
      else if (rank > 900000 && rank <= 1000000) {
        return '120%'
      }
      else if (rank > 800000 && rank <= 900000) {
        return '140%'
      }
      else if (rank > 700000 && rank <= 800000) {
        return '160%'
      }
      else if (rank > 600000 && rank <= 700000) {
        return '180%'
      }
      else if (rank > 500000 && rank <= 600000) {
        return '200%'
      }
      else if (rank > 400000 && rank <= 500000) {
        return '240%'
      }
      else if (rank > 300000 && rank <= 400000) {
        return '280%'
      }
      else if (rank > 200000 && rank <= 300000) {
        return '340%'
      }
      else if (rank > 100000 && rank <= 200000) {
        return '400%'
      }
      else if (rank > 90000 && rank <= 100000) {
        return '500%'
      }
      else if (rank > 80000 && rank <= 90000) {
        return '600%'
      }
      else if (rank > 70000 && rank <= 80000) {
        return '700%'
      }
      else if (rank > 60000 && rank <= 70000) {
        return '700%'
      }
      else if (rank > 50000 && rank <= 60000) {
        return '800%'
      }
      else if (rank > 40000 && rank <= 50000) {
        return '900%'
      }
      else if (rank > 30000 && rank <= 40000) {
        return '1000%'
      }
      else if (rank > 20000 && rank <= 30000) {
        return '1100%'
      }
      else if (rank > 10000 && rank <= 20000) {
        return '1200%'
      }
      else if (rank <= 10000) {
        return '3000%'
      }
    },
    category (nodes) {
      var node = nodes.BrowseNode[0]
      if (!node) {
        return
      }
      var resultArray = [node.Name]
      var Ancestors = node.Ancestors
      while (Ancestors) {
        if (Ancestors.BrowseNode.IsCategoryRoot !== '1') {
          resultArray.unshift(Ancestors.BrowseNode.Name)
        }
        Ancestors = Ancestors.BrowseNode.Ancestors
      }
      return resultArray.join(' > ')
    },
    reset () {
      this.productList = []
      this.offset = 0
    },
    search (done) {
      this.getData(this.title, this.editorial, this.feature, LIMIT, this.offset, done)
    },
    refresher: function (index, done) {
      // console.log('refresher', this.offset)
      this.search(done)
    },
    getData (title, editorial, feature, limit, offset, done) {
      if (title === '') {
        title = null
      }
      if (editorial === '') {
        editorial = null
      }
      if (feature === '') {
        feature = null
      }
      console.log(offset, title, editorial, feature)
      axios.get(HOST, {
        params: {
          title: title,
          editorial: editorial,
          feature: feature,
          limit: limit,
          offset: offset
        }
      }).then(response => {
        if (done) {
          done()
        }
        if (response.data.status === 'success') {
          this.productList = this.productList.concat(response.data.results)
          // console.log('length', this.productList.length)
          this.offset = offset + LIMIT
        }
        else {
          Alert.create({
            enter: 'bounceInRight',
            leave: 'bounceOutRight',
            color: 'positive',
            icon: 'wifi',
            html: response.data.msg,
            position: 'top-right',
            actions: [
              {
                label: 'Abort'
              }
            ]
          })
        }
      })
    },
    highlight (content, keyword) {
      return content.replace(keyword, '<span class="highlight">' + keyword + '</span>')
    },
    dislike (_id) {
      axios.put(HOST + '/dislike', {
        _id: _id
      }).then(response => {
        if (response.data.status === 'success') {
          console.log('dislike success')
          Toast.create['info']({
            html: 'dislikeしました',
            timeout: 1000
          })
        }
        else {
          Alert.create({
            enter: 'bounceInRight',
            leave: 'bounceOutRight',
            color: 'positive',
            icon: 'wifi',
            html: response.data.msg,
            position: 'top-right',
            actions: [
              {
                label: 'Abort'
              }
            ]
          })
        }
      })
    },
    favorite (_id, index) {
      axios.put(HOST + '/favorite', {
        _id: _id
      }).then(response => {
        if (response.data.status === 'success') {
          this.productList[index].favorite = true
          Toast.create({
            html: 'お気に入り追加しました',
            timeout: 1000
          })
        }
        else {
          Alert.create({
            enter: 'bounceInRight',
            leave: 'bounceOutRight',
            color: 'positive',
            icon: 'wifi',
            html: response.data.msg,
            position: 'top-right',
            actions: [
              {
                label: 'Abort'
              }
            ]
          })
        }
      })
    }
  },
  // mounted: function () {
  //   console.log(this)
  //   // this.getData(10)
  // },
  components: {
    QCard,
    QCardTitle,
    QCardMain,
    QCardActions,
    QCardMedia,
    QCardSeparator,
    QBtn,
    QIcon,
    QModal,
    QTooltip,
    QInfiniteScroll,
    QSpinnerDots,
    QInput,
    QRating
  }
}
</script>

<style>
  .intro {
    overflow-y: scroll;
    max-height: 40vh;
  }
  .highlight {
    background-color: yellow
  }
  .detail:hover {
    cursor: pointer;
  }
</style>
