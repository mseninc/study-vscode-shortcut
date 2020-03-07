<template>
  <q-page class="q-pa-md">
    <!-- Question card -->
    <q-card>
      <q-card-section>
        <q-input outlined bottom-slots readonly autogrow class="text-h6 q-mb-lg"
          input-class="question-textbox"
          :value="question.question">
          <template v-slot:hint>
            <div class="text-body1">
              {{ $t('training.label.questionPosition', [current + 1, total]) }}
            </div>
          </template>
          <template v-slot:after>
            <q-btn round icon="volume_up" @click="speakText" :disable="isWorking" />
          </template>
        </q-input>
        <div v-if="question.meanings" class="row items-center">
          <div class="col col-auto">
            <q-toggle :label="$t('training.label.meanings')"
              v-model="showMeanings" class="q-mr-sm meanings-toggle" />
          </div>
          <div class="col">
            <q-input v-if="showMeanings"
              dense outlined readonly autogrow class="text-body1"
              input-class="meanings-textbox"
              :value="question.meanings">
            </q-input>
          </div>
        </div>
      </q-card-section>
    </q-card>
    <!-- Answer input -->
    <div class="row q-mt-md">
      <div class="col col-12">
        <q-input outlined clearable class="text-h6"
          ref="yourAnswerInput"
          :readonly="!$q.platform.is.ios"
          v-model="yourAnswer"
          @focus="resetAnswer">
          <template v-slot:prepend>
            <q-icon size="32px" name="record_voice_over" :title="$t('training.label.yourAnswer')" />
          </template>
          <template v-slot:after>
            <q-btn round color="teal" icon="play_arrow"
              :title="$t('training.label.speakYourAnswer')"
              :disable="isWorking || !yourAnswer"
              @click="speakYourAnswer"
              />
          </template>
        </q-input>
      </div>
    </div>
    <div class="row justify-center items-center q-mt-lg">
      <!-- Voice input button for Android, PC -->
      <voice-input-button
        class="q-mr-lg"
        v-show="!$q.platform.is.ios"
        :lang="language.code"
        :disabled="isWorking"
        @start="isWorking = true"
        @end="isWorking = false"
        @success="answer"
        @error="failedToRecognize"
        />
      <!-- Voice input button for iOS -->
      <q-btn
        class="q-mr-lg"
        color="teal" round size="32px" icon="mic"
        v-show="$q.platform.is.ios"
        @click="$refs.yourAnswerInput.focus()"
        />
      <!-- Skip button -->
      <q-btn color="negative" round size="24px"
        :label="$t('training.label.skip')"
        :title="$t('training.label.skip')"
        @click="$emit('skip')"/>
    </div>
    <div class="row justify-center q-mt-lg">
      <challenge-quit-button @quit="$emit('quit')" />
    </div>
  </q-page>
</template>

<style lang="stylus" scoped>

.q-toggle.meanings-toggle
  margin-left -8px

.q-field .question-textbox
  line-height 1.5
  padding 10px 0px

.q-field .meanings-textbox
  line-height 1.5
  padding 5px 0px

</style>

<script>
import { speak, sanitize } from 'boot/speech';
import VoiceInputButton from 'components/training/VoiceInputButton';
import ChallengeQuitButton from 'components/training/ChallengeQuitButton';

export default {
  components: {
    VoiceInputButton,
    ChallengeQuitButton,
  },
  props: {
    question: Object,
    current: Number,
    total: Number,
    language: Object,
  },
  data() {
    return {
      showMeanings: false,
      isWorking: false,
      yourAnswer: '',
      iosInputTimer: null,
      iosHasInput: false,
    };
  },
  computed: {
  },
  methods: {
    async speakText() {
      this.isWorking = true;
      try {
        await speak({
          text: this.question.speech_text || this.question.question,
          lang: this.language.code,
        });
        this.$emit('replayExpectedAnswer');
      } catch (error) {
        this.$notifyError({ message: this.$t('message.failedSpeechSynthesis'), icon: 'warning', error });
      } finally {
        this.isWorking = false;
      }
    },
    async speakYourAnswer() {
      if (!this.yourAnswer) return;
      this.isWorking = true;
      try {
        await speak({
          text: this.yourAnswer,
          lang: this.language.code,
        });
        this.$emit('replayOwnAnswer');
      } catch (error) {
        this.$notifyError({ message: this.$t('message.failedSpeechSynthesis'), icon: 'warning', error });
      } finally {
        this.isWorking = false;
      }
    },
    /**
     * Checks if input answer is correct or not
     */
    checkAnswer(answer) {
      // Sanitize both answers to avoid false-positive
      const correctAnswer = sanitize(this.question.correct_answer);
      return sanitize(answer) === correctAnswer;
    },
    /**
     * Called from VoiceInputButton.success
     */
    answer({ content, confidence }) {
      this.$emit('try');
      const isCorrect = this.checkAnswer(content);
      // Display input answer but correct_answer if the actual answer is correct
      this.yourAnswer = isCorrect ? this.question.correct_answer : content;
      this.$emit('answer', { content, confidence, isCorrect });
    },
    /**
     * Called from VoiceInputButton.error
     */
    failedToRecognize(e) {
      const messageKey = e.error === 'not-allowed' ? 'recognitionNotAllowed' : 'failedToRecognize';
      this.$notifyError({ message: this.$t(`message.${messageKey}`), icon: 'warning', error: e });
    },
    /**
     * Resets answering states
     */
    resetAnswer() {
      this.yourAnswer = '';
      this.iosHasInput = false;
    },
  },
  mounted() {
  },
  watch: {
    question() {
      this.resetAnswer();
    },
    yourAnswer(content) {
      if (this.iosInputTimer) clearTimeout(this.iosInputTimer);
      if (!this.$q.platform.is.ios) return; // Exit if not iOS
      if (!content) return; // Exit if text is nothing
      if (this.iosHasInput) return; // Exit if answer has been input
      // If iOS, emit answer event when inputting has been stopped for 1 sec
      this.iosInputTimer = setTimeout(() => {
        this.iosHasInput = true; // Restrict following event to avoid redundant answering
        this.answer({ content });
        this.$refs.yourAnswerInput.blur();
      }, 1000);
    },
  },
};
</script>
