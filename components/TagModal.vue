<template>
  <div id="tag-modal"
       class="h-0 w-100p absolute b-0 l-0 no-display column bg-color-black
              align-center justify--start clip-overflow--x">
    <h2 class="mt-64">Pick a color</h2>
    <ColorsCarousel :current-color="currentTag?.color" />
    <input v-model="name" class="m-30-0 p-15-20 h-20 w-100 min-h-20 max-h-40 no-resize
                                 color-white center-text bg-color-secondary-black focus:no-outline"
           maxlength="30" placeholder="Name" autocomplete="off" spellcheck="false"
           border="none rad-10" font="s-1rem" required="true">
    <button v-if="currentTag" class="p-8-16 pointer bg-color-white"
            border="none rad-8" font="s-1em" @click="updateTag">Update</button>
    <button v-else class="p-8-16 pointer bg-color-white" border="none rad-8"
            font="s-1em" @click="addTag">Add</button>
    <a class="pointer absolute -t-64 r-32 z-2 color-white bg-color-black p-2"
       font="s-2.5em fam-monospace" @click="closeModal">x</a>
  </div>
</template>

<script lang="ts">
import { gsap } from 'gsap';
import { mapWritableState, mapActions } from 'pinia';
import { useTagsStore } from '~/stores/tagsStore';
import { db } from '~/db';

export default defineNuxtComponent({
  data: () => ({
    name: '',
  }),

  computed: {
    color() {
      const activeSlide = document.querySelector('#tag-modal .swiper-slide-active') as Element;
      const styles = getComputedStyle(activeSlide);
      return styles.getPropertyValue('--color');
    },

    ...mapWritableState(useTagsStore, ['tags', 'showModal', 'currentTag']),
  },

  mounted() {
    if (this.currentTag) {
      this.name = this.currentTag.name;
    }
  },

  methods: {
    closeModal() {
      gsap.to('#tag-modal', {
        height: '0',
        display: 'none',
        duration: 0.5,
        onComplete: () => {
          this.currentTag = null;
          this.showModal = false;
        },
      });
    },

    async addTag() {
      await db.tags.add({ name: this.name, color: this.color });
      await this.updateTags();
      this.closeModal();
    },

    async updateTag() {
      if (this.currentTag) {
        await db.tags.add({ id: this.currentTag.id, name: this.name, color: this.color });
        await this.updateTags();
        this.closeModal();
      } else {
        this.addTag();
      }
    },

    ...mapActions(useTagsStore, ['updateTags']),
  },
});
</script>
