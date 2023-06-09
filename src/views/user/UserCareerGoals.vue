<script setup lang="ts">
  import { watchEffect, ref, computed } from 'vue';
  import CareerGoal from '@/types/CareerGoal';
  import { urlCareerGoalData } from '@/components/constants/urlConstants';
  import { getJSON } from '@/utils/requestUtils';
  import LoadingSpinner from '@/components/widgets/LoadingSpinner.vue';
  import Notification from '@/components/widgets/Notification.vue';

  // define colors for donut
  const colorDonutForeground = '#E8E9EB';
  const colorDonutSection = '#493DF5';

  const latestCareerGoal = ref(new CareerGoal);
  const hasData = ref(false);
  const error = ref('');
  const isLoading = ref(false);
  
  const loadData = async () => {
    isLoading.value = true;
      try {
        const response = await getJSON(urlCareerGoalData);
        // there can be multiple goals for a user but we only want to show the lastest / newest
        const maxId = Math.max.apply(null, response.data.map((item: { id: any; }) => item.id));
        const latest = response.data.find( (item: { id: number; }) => {return item.id === maxId})
        latestCareerGoal.value.populateFromJSON(latest);
        hasData.value = !!latestCareerGoal.value.data || false;
      } catch (e) {
        error.value = 'We are sorry - there was an error loading your career goal data.'
      }
  }

  watchEffect(async (newGoal) => {
    if (newGoal.length > 0) {
      await loadData()
    }
    isLoading.value = false;
  });

  const getProgress = computed(() => latestCareerGoal.value.data.progress || 0)

  // make refs visible for tests
  defineExpose({ latestCareerGoal, hasData, error, isLoading, getProgress });
</script>

<template>
  <div class="career-goal">
    <h4>Career Goal</h4>
    <div v-if="isLoading">
      <LoadingSpinner />
    </div>

    <div v-else class="career-goal--wrapper">
      <div v-if="error !== ''" class="career-goal">
        <Notification :msg="error"/>
      </div>

      <template v-if="hasData">
        <div class="career-goal--header">Your Progress</div>
        <div class="career-goal--donut">
          <vc-donut 
            :foreground="colorDonutForeground"
            :sections="[{ value: getProgress, color: colorDonutSection }]"
            :size="180"
            :thickness="14"
            ><div class="career-goal--donut-text">{{ getProgress }}%</div></vc-donut>
        </div>
        <div>I want to become a</div>
        <h4 class="career-goal--name">{{ latestCareerGoal.data.name }}</h4>

        <div class="career-goal--link">View Insights</div>
      </template>
      <template v-else >
        <div class="career-goal--error">Nothing here yet</div>
      </template>
    </div>
  </div>
</template>

<style scoped>
  h4 {
    font-size: var(--font-size-h4);
    font-weight: var(--font-weight-bold);
    line-height: 32px;
  }
  .career-goal {
    min-width: 262px;
  }
  .career-goal--wrapper {
    border: 1px solid var(--color-border);
    margin-top: 16px;
    padding: 32px 20px;
    border-radius: 6px;
    font-size: var(--font-size-small);
    text-align: center;
  }
  .career-goal--header {
    font-weight: var(--font-weight-bold);
    color: var(--color-text-light);
  }
  .career-goal--donut {
    margin: 24px 20px;
  }
  .career-goal--donut-text {
    font-size: var(--font-size-h2);
    color: var(--color-donut-text);
    font-weight: var(--font-weight-bold);
  }
  .career-goal--link {
    margin-top: 24px;
    margin-bottom: 13px;
    color: var(--color-background-user-initials);
    font-weight: var(--font-weight-bold);
  }
  .career-goal--error {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
</style>