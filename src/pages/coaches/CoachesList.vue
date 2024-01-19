<template>
  <div>
    <base-dialog :show="!!error" title="An error occured." @close="handleError">
      <p>{{ error }}</p>
    </base-dialog>
    <section>
      <coach-filter @change-filter="setFilter"></coach-filter>
    </section>
    <section>
      <base-card>
        <div class="controls">
          <base-button mode="outline" @click="loadCoaches(true)">Refresh</base-button>
          <base-button v-if="!isCoach && !this.isLoading" link to="/register"
            >Register as a Coach</base-button
          >
        </div>
        <div v-if="isLoading">
          <base-spinner></base-spinner>
        </div>
        <ul v-else-if="hasCoaches">
          <coaches-item
            v-for="coach in filteredCoaches"
            :key="coach.id"
            :id="coach.id"
            :firstName="coach.firstName"
            :lastName="coach.lastName"
            :rate="coach.hourlyRate"
            :areas="coach.areas"
          ></coaches-item>
        </ul>
        <h3 v-else>No coaches found.</h3>
      </base-card>
    </section>
  </div>
</template>

<script>
import CoachesItem from '../../components/coaches/CoachesItem.vue';
import CoachFilter from '../../components/coaches/CoachFilter.vue';
export default {
  components: { CoachesItem, CoachFilter },
  data() {
    return {
      isLoading: false,
      error: null,
      activeFilters: {
        frontend: true,
        backend: true,
        career: true
      }
    };
  },
  created() {
    this.loadCoaches();
  },
  methods: {
    async loadCoaches(refresh = false) {
      this.isLoading = true;
      try {
        await this.$store.dispatch('coaches/loadCoaches', { forceRefresh: refresh });
      } catch (error) {
        this.error = error.message || 'Something went wrong!';
      }

      this.isLoading = false;
    },
    setFilter(updatedFilters) {
      this.activeFilters = updatedFilters;
    },
    handleError() {
      this.error = null;
    }
  },
  computed: {
    filteredCoaches() {
      const coaches = this.$store.getters['coaches/coaches'];
      return coaches.filter((coach) => {
        if (this.activeFilters.frontend && coach.areas.includes('frontend')) {
          return true;
        }
        if (this.activeFilters.backend && coach.areas.includes('backend')) {
          return true;
        }
        if (this.activeFilters.career && coach.areas.includes('career')) {
          return true;
        }
        return false;
      });
    },
    hasCoaches() {
      return !this.isLoading && this.$store.getters['coaches/hasCoaches'];
    },
    isCoach() {
      return this.$store.getters['coaches/isCoach'];
    }
  }
};
</script>

<style scoped>
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.controls {
  display: flex;
  justify-content: space-between;
}
</style>
