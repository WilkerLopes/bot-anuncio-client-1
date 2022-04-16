<template>
  <ion-page id="ViewAdPage">
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/" color="dark" text=""></ion-back-button>
        </ion-buttons>

        <ion-title>
          {{ ads ? ads.title : "Carregando ..." }}
        </ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="load" v-if="isLoading">
        <section>
          <div class="container">
            <div class="header-banner">
              <ion-skeleton-text animated style="width: 100%; height: 350px"></ion-skeleton-text>
            </div>

            <div class="content">
              <div class="card-superior-info">
                <div class="title">
                  <ion-skeleton-text animated style="width: 250px; height: 24px"></ion-skeleton-text>
                </div>
                <div class="price">
                  <ion-skeleton-text animated style="width: 70px; height: 24px"></ion-skeleton-text>
                </div>
              </div>

              <div class="locale"><ion-skeleton-text animated style="width: 200px; height: 16px"></ion-skeleton-text></div>

              <div class="title-page left-text small"><ion-skeleton-text animated style="width: 150px; height: 16px"></ion-skeleton-text></div>
              <div class="container-box">
                <ion-skeleton-text animated style="width: 100px; height: 32px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 60px; height: 32px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 120px; height: 32px"></ion-skeleton-text>
              </div>

              <div class="descriprion">
                <ion-skeleton-text animated style="width: 95%; height: 24px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 98%; height: 24px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 97%; height: 24px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 100%; height: 24px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 99%; height: 24px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 96%; height: 24px"></ion-skeleton-text>
                <ion-skeleton-text animated style="width: 60%; height: 24px"></ion-skeleton-text>
              </div>
            </div>
          </div>
        </section>
      </div>

      <div v-if="ads && !isLoading">
        <section>
          <div class="container">
            <div class="header-banner">
              <Splide :options="imagesOptions">
                <SplideSlide v-for="(image, index) in ads.images" :key="index">
                  <img :src="image" @click="expandImage(image)" />
                </SplideSlide>
              </Splide>
            </div>
          </div>
        </section>

        <!--
        <ion-modal :is-open="isExpand" css-class="modal-imagem-expand" @didDismiss="isExpand = false">
          <ion-content>
            <img :src="imageExpand ? imageExpand : '/assets/logo/favicon-16x16.png'" />
          </ion-content>
        </ion-modal>
-->
        <section>
          <div class="container">
            <div class="content">
              <div class="card-superior-info">
                <div class="title">{{ ads.title }}</div>
                <div class="price" v-if="ads.type != 'donate' || ads.typeAd != 'recommendation' || ads.typeAd != 'notice'">{{ printMoney(ads.price, ads.type) }}</div>
                <div class="type" v-else>{{ isTypeTransaction(ads.type) }}</div>
              </div>

              <div class="locale">{{ getAdress(ads.locale) }}</div>

              <div class="title-page left-text small">Pagamentos Aceitos:</div>
              <div class="container-box">
                <div class="box" v-for="(item, index) in ads.paymentAccepted" :key="index">{{ isPaymentAccepted(item) }}</div>
              </div>

              <div class="descriprion">
                <p>{{ ads.description }}</p>
              </div>
            </div>
          </div>
        </section>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { defineComponent } from "vue";
import { mapActions } from "vuex";
import { Splide, SplideSlide } from "@splidejs/vue-splide";
import "@splidejs/splide/dist/css/themes/splide-skyblue.min.css";

export default defineComponent({
  components: { Splide, SplideSlide },
  name: "ViewAdPage",
  data() {
    return {
      imagesOptions: {
        speed: 400,
        height: "350px",
        autoWidth: true,
        gap: 24,
        arrows: false,
        pagination: true,
        focus: 1,
        trimSpace: false,
        easing: "cubic-bezier(0.25, 1, 0.5, 1)",
        breakpoints: {
          780: {
            height: "200px",
          },
          1024: {
            height: "250px",
          },
        },
      },
      ads: null,
      isExpand: false,
      imageExpand: null,
      isLoading: false,
    };
  },
  async mounted() {
    this.isLoading = true;
    let req = await this.getAdsById(this.$route.params.id);
    if (req.status === 404) await this.$router.replace({ name: "NotFound", params: { type: "ads" } });
    if (req.data.active !== "approved") await this.$router.replace({ name: "NotFound", params: { type: "ads" } });

    this.ads = req.data;
    if (this.ads.active === "approved") {
      this.isLoading = false;
      this.moreViews(this.$route.params.id);
    }
  },
  methods: {
    ...mapActions("ad", ["getAdsById", "moreViews"]),
    getAdress(locale) {
      return `${locale.logradouro}, ${locale.bairro} - ${locale.localidade}`;
    },
    expandImage(image) {
      this.imageExpand = image;
      this.isExpand = true;
    },
  },
});
</script>

<style lang="scss" scoped>
#ViewAdPage {
  ion-title {
    text-align: center;
    &::first-letter {
      text-transform: uppercase;
    }
  }

  .header-banner {
    border-radius: 12px;
    width: 100%;
    margin: 32px 0 64px;

    .splide__track {
      border-radius: 12px;
      overflow: hidden;
    }
    img {
      cursor: pointer;
      height: 100%;
      margin: 0 auto;
      border-radius: 12px;
    }
  }

  .container-box {
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .title-page {
    margin-bottom: 8px;
  }
  .box {
    border: 1px solid rgba(var(--ion-color-medium-rgb), 0.35);
    box-sizing: border-box;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 1;
    transition: all 0.35s;
    width: auto;
    font-size: 0.8em;
    color: var(--ion-color-primary);
    padding: 8px 16px;
  }

  .content {
    z-index: 9;
    flex-grow: 1;
    .card-superior-info {
      display: flex;
      align-items: flex-start;
      justify-content: space-between;
      gap: 8px;
      .title {
        font-family: "Mulish", sans-serif;
        font-size: 1.5em;
        line-height: 100%;
        font-weight: 800;
        &::first-letter {
          text-transform: uppercase;
        }
      }

      .price {
        color: var(--ion-color-primary);
        font-size: 1.2em;
        font-family: "Mulish";
        font-style: normal;
        font-weight: 700;
        line-height: 100%;
        opacity: 0.72;
        text-align: right;
      }

      .type {
        background: var(--ion-color-secondary-tint);
        padding: 4px 8px;
        text-transform: capitalize;
        color: var(--ion-color-secondary-contrast);
        font-size: 1.1em;
        font-family: "Mulish" !important;
        font-style: normal;
        font-weight: 700;
        line-height: 100%;
        text-align: right;
        border-radius: 6px;
      }
    }
    .locale {
      font-family: "Questrial", sans-serif !important;
      font-size: 1em;
      font-weight: 400;
      text-transform: unset;
      opacity: 0.8;
      color: var(--ion-color-medium);
      line-height: 100%;
      margin-top: 8px;
      margin-bottom: 16px;
    }
    .descriprion {
      font-size: 1.1em;
      line-height: 125%;
      color: var(--ion-text-color);
      opacity: 0.9;
      margin-top: 24px;
      p {
        margin: 0;
        &::first-letter {
          text-transform: uppercase;
        }
      }
    }
  }
}

.modal-imagem-expand {
  --background: transparent !important;
  --width: 90%;
  --max-width: 1024px;
  --min-height: 50vh;
  --max-height: 80vh;
  padding: 0 !important;
  display: flex;
  align-items: center;
  justify-content: center;
  img {
    width: 100%;
    border-radius: 12px;
  }
}
</style>