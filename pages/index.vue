<template>
	<section class="flex flex-col flex-1 lg:flex-row lg:flex-none lg:mt-16 sm:gap-x-10 md:gap-x-20">
		<div class="flex flex-col w-full lg:w-1/2">
			<Profile />
			<CompletedChallenges />
      <Countdown @completed="getNewChallenge" />
      <button v-if="hasCountdownCompleted" disabled class="button completed">
        Ciclo Completado
      </button>
      <button v-else-if="isCountdownActive" class="button abandon" @click="setCountdownState(false)">
        Abandonar Ciclo
      </button>
      <button v-else class="button start" @click="setCountdownState(true)">
        Começar um ciclo
      </button>
    </div>
    <Card id="challenge" class="w-full lg:w-1/2" />
  </section>
</template>

<script lang="ts">
import Vue from 'vue';

import { mapState, mapMutations, mapGetters } from 'vuex';
import { Mutations as ChallengesMT } from '~/store/Challenges/types'
import { Mutations as CountdownMT } from '~/store/Countdown/types'

import Card from '~/components/organisms/Card.vue';
import CompletedChallenges from '~/components/atoms/CompletedChallenges.vue';
import Profile from '~/components/molecules/Profile.vue';
import Countdown from '~/components/molecules/Countdown.vue';

import {
  scrollToElement,
  getRandomNumber,
  playAudio,
  sendNotification,
} from '~/utils';

interface Head {
  title: string,
}

export default Vue.extend({
  head (): Head {
    return {
      title: 'Home | movue.it',
    }
  },
  components: {
    Card,
    CompletedChallenges,
    Profile,
    Countdown,
  },
  mounted() {
    if ('Notification' in window) {
      Notification.requestPermission();
    }
  },
  computed: {
    ...mapState('Countdown', {
      hasCountdownCompleted: 'hasCompleted',
      isCountdownActive: 'isActive',
    }),
    ...mapGetters('Challenges', ['challengesLength']),
  },
  methods: {
    ...mapMutations({
      setCountdownHasCompleted: `Countdown/${CountdownMT.SET_HAS_COMPLETED}`,
      setCountdownIsActive: `Countdown/${CountdownMT.SET_IS_ACTIVE}`,
      setCurrentChallengeIndex: `Challenges/${ChallengesMT.SET_CURRENT_CHALLENGE_INDEX}`
    }),
    setCountdownState (flag: boolean) {
      this.setCountdownHasCompleted(false);
      this.setCountdownIsActive(flag);
    },
    getNewChallenge () {
      const index = getRandomNumber(0, this.challengesLength);
      
      this.setCountdownHasCompleted(true);
      this.setCurrentChallengeIndex(index);
      
      if (Notification?.permission === 'granted') {
        playAudio('/notification.mp3');
        sendNotification('Novo Desafio', {
          body: 'Um novo desafio começou, vá completa-lo!',
          icon: '/favicon.png'
        })
      }

      this.$nextTick(() => {
        scrollToElement('#challenge');
      });
    }
  }
});
</script>
