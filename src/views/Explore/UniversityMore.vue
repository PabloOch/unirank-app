<template>
    <ion-page>
        <header-section />
        <ion-content :fullscreen="true" class="ion-padding-top">
            <ion-grid style="display:flex; flex-flow: column; justify-content: center; margin-top: 12%;">
                <ion-row class="ion-padding-top">
                    <ion-col class="scrolling">
                        <div v-for="tabTitle in tabTitles" :key="tabTitle">
                            <ion-card :class="btpos === tabTitle? 'selected-tab' : 'more-tab'" @click="changeTabColor(tabTitle)">
                                <p :class="btpos === tabTitle?'selected-tab-title':'tab-title'">{{ $t(`${tabTitle}`) }}</p>
                            </ion-card>
                        </div>
                    </ion-col>
                </ion-row>
                <ion-row style="padding-left: 3px;">
                    <ion-col style=" display: flex; flex-flow: row; ">
                        <ion-text>
                            <p class="title-name" style="margin:0px">{{ $t(`${btpos} ${datas.length}`) }}</p>
                        </ion-text>
                    </ion-col>
                </ion-row>
                <ion-row v-for="data in datas" :key="data.id">
                    <ion-card style="margin:10px 8px 0 10px;" @click="toUniversityDetailModal(data.id)">
                        <ion-card-content>
                            <div style="display: flex; flex-flow: row;">
                                <ion-img :src='data.logo_url' class="leftImg" style="width: 15%;"/>
                                <div style="display: flex; flex-flow: column; margin-top: 3%; margin-left:1px; width: 85%;">
                                    <div>
                                        <p class="university-name" style="float:left;">{{ short_name(data.university_name) }}</p>
                                        <ion-img :src='nextImgUrl' class="next-img" style="float:right;  margin-top: 6px;"></ion-img>
                                    </div>
                                    <p class="university-ranking">{{ $t(`Local #${data.local_position} | Global #${data.global_position} | Score ${data.score}`) }}</p>
                                    <div>
                                        <p class="university-country" style="float: left;">{{ $t(`data.country`) }}</p>
                                        <p class="university-status" style="float: right">{{ $t(`data.status`) }}</p>
                                    </div>
                                </div>
                            </div>
                        </ion-card-content>
                    </ion-card>
                </ion-row>
            </ion-grid>
        </ion-content>
        <filter-footer-section />
    </ion-page>
</template>

<script>
import {useExploreDataStore} from "@/store";
import {
    IonPage,
    IonContent,
    IonGrid,
    IonRow,
    IonCol,
    IonCard,
    IonCardContent,
    IonText,
    IonImg,
    modalController,
} from "@ionic/vue";
import {computed, defineComponent, ref, onBeforeMount} from "vue";
import {useRouter} from "vue-router";
import {useLoadingStore} from "@/store/loading";
import HeaderSection from "@/components/explore/HeaderSection.vue";
import FilterFooterSection from "@/components/explore/FilterFooterSection.vue";
import UniversityDetailModal from "@/components/modal/UniversityDetailModal.vue";

export default defineComponent({
  name: "UniversityMore",
  components: {
    HeaderSection,
    FilterFooterSection,
    IonPage,
    IonContent,
    IonGrid,
    IonRow,
    IonCol,
    IonCard,
    IonCardContent,
    IonText,
    IonImg,
  },
  setup() {
    const store = useExploreDataStore();
    const {loadEliteData, changeLoadedVal} = store;
    const dataLoaded = computed(() => store.elite_dataLoaded);
    const elite_datas = computed(() => store.elite_datas);
    const world_top_datas = computed(() => store.world_top_datas);
    const region_top_datas = computed(() => store.region_top_datas);
    const country_top_datas = computed(() => store.country_top_datas);
    const verified_datas = computed(() => store.verified_datas);
    const {showLoading, hideLoading} = useLoadingStore();
    const router = useRouter();

    const tabTitles = ['Elite Institues', 'World Top Institutes', ' Top Region Institutes', 'Top Country Institutes', 'Verified Institutes'];
    const imgUrl = '/assets/images/header.svg';
    const nextImgUrl = 'assets/images/Chevron.svg';
    // const maxNameLength = 32;
    const btpos = ref(tabTitles[0]);
    const datas = ref([]);

    onBeforeMount(() => {
      if (!dataLoaded.value) {
        showLoading();
        Promise.all([loadEliteData()]).then(() => {
            datas.value = elite_datas.value;
            hideLoading();
            changeLoadedVal();
        })
      }
    });

    const short_name = (name) => {
        let newname = name;
        if (name.length>32) {
            newname = name.substring(0, 32);
            newname = `${newname}...`;
        }
        return newname;
    }

    const changeTabColor = (title) => {
        btpos.value = title;
        if(title === tabTitles[0])
            datas.value = elite_datas.value
        if(title === tabTitles[1])
            datas.value = world_top_datas.value
        if(title === tabTitles[2])
            datas.value = region_top_datas.value
        if(title === tabTitles[3])
            datas.value = country_top_datas.value
        if(title === tabTitles[4])
            datas.value = verified_datas.value

        console.log(datas.value);
    }    
    const toUniversityDetailModal = async(id) => {
      const modal = await modalController.create({
          component: UniversityDetailModal,
          componentProps: {
              id:id
          },
          initialBreakpoint: 0.95,
          breakpoints: [0, 0.95],
        });
        modal.present();
    }

    return {
        imgUrl,
        nextImgUrl,

        tabTitles,
        dataLoaded,
        datas,
        btpos,

        short_name,
        changeTabColor,
        toUniversityDetailModal,
    };
  },
});
</script>
<style scoped>
ion-content {
    --background: #ffffff;
}
ion-text {
    padding:0%;
}
ion-card {
    width: 100%;
    height: auto;
    border-radius: 6px;
    filter: drop-shadow(0px 3px 3px rgba(0,0,0,0.16 ));
    background: #ffffff;
}
ion-card-content {
    padding-top:5px;
}
.more-tab {
    width: max-content;
    height: 36px;
    border-radius: 15px;
    margin: 0px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-right: 10px;
    padding-left: 10px;
    padding-right: 10px;
}
.tab-title{
    background: #ffffff;
    font-family: "Calibri";
    font-size: 20px;
    font-weight: normal;
    font-style: normal;
    color: #1c345a;
    text-align: center;
}
.title-name {
    font-family: "Calibri";
    font-size: 16px;
    font-weight: normal;
    font-style: normal;
    text-align: left;
    color: #1c345a;
}
.leftImg {
    width: 50px;
    height: 50.6px;
    margin-top: 5%;
}
.university-name {
    font-family: "Calibri";
    font-size: 16px;
    font-weight: normal;
    font-style: normal;
    text-align: left;
    color: #203456;
    margin-right: 10%;
}
.university-ranking {
    font-family: "Calibri";
    font-size: 12px;
    font-weight: 300;
    font-style: normal;
    text-align: left;
    color: #203456;
}
.university-country{
    font-family: "Calibri";
    font-size: 12px;
    font-weight: normal;
    font-style: normal;
    color: #606060;
}
.university-status {
    font-family: "Calibri";
    font-size: 12px;
    font-weight: normal;
    font-style: normal;
    color: #007a00;
}
.next-img {
    width: 7.7px;
}
.scrolling {
    display: flex; 
    flex-flow: row;
    overflow-x: scroll;
    overflow-y: hidden;
}
.scrolling::-webkit-scrollbar {
    display: none;
}
.selected-tab {
    width: max-content;
    height: 36px;
    border-radius: 15px;
    margin: 0px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-right: 10px;
    padding-left: 10px;
    padding-right: 10px;
    background: #1c345a
}
.selected-tab-title {
    font-family: "Calibri";
    font-size: 20px;
    font-weight: normal;
    font-style: normal;
    text-align: center;
    color: #ffffff;
}
</style>