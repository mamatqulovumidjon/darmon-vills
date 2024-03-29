<template>
  <page-layout
      :page-title="pageTitle"
  >
    <card-list>
      <house-list-item
          v-for="house in houses"
          :key="house.id"
          :house="house"
          show-by-order
      />
      <card-list-item
          :active="isAllActive"
          @click="switchSectionLights"
      >
        Все
      </card-list-item>
    </card-list>
  </page-layout>
</template>

<script lang="ts">
import { useRoute, useRouter } from 'vue-router';
import { getSection, getSectionHouses, House, Section } from '@/data/sections';
import { defineComponent } from 'vue';
import { HOME_PAGE_URL } from "@/utils/routes";
import HouseListItem from "@/components/house/HouseListItem.vue";
import PageLayout from "@/components/page/PageLayout.vue";
import CardList from "@/components/list/CardList.vue";
import { getEmptyLights, Lights } from '@/data/lights'
import CardListItem from '@/components/list/CardListItem.vue'
import { mapActions, mapGetters, mapState } from 'vuex'

type SetupData = {
  section: Section
  houses: House[]
  isAllActive: boolean
}

export default defineComponent({
  name: 'SectionPage',
  components: {
    CardListItem,
    CardList,
    PageLayout,
    HouseListItem
  },
  setup (): SetupData | never {
    const route = useRoute();
    const section = getSection(parseInt(route.params.section as string));

    if (!section) {
      const router = useRouter();
      router.push(HOME_PAGE_URL);
    }

    return {
      isAllActive: false,
      section,
      houses: getSectionHouses(section!.id)
    } as SetupData
  },
  computed: {
    ...mapState({
      lights: 'lights'
    }),
    pageTitle (): string {
      return `Секция: ${this.section.name}`
    }
  },
  mounted () {
    this.isAllActive = this.houses.every(house => {
      return (house.row in this.lights) && this.lights[house.row].includes(house.col)
    })
  },
  methods: {
    ...mapActions({
      updateLights: 'updateLights'
    }),
    switchSectionLights () {
      const lights: Lights = getEmptyLights()
      this.isAllActive = !this.isAllActive

      if (this.isAllActive) {
        this.houses.forEach((house: House) => {
          lights[house.row].push(house.col)
        })
      }

      this.updateLights(lights)
    }
  }
});
</script>
