<!-- Window is fixed, 102px, pointer cursor, gradual blurry effect on surrounding words. -->
<!--  Comprehension questions appear afterwards in the same slide -->

<template>
  <Experiment title="Mouse tracking for Reading" translate="no">

    <Screen :title="'welcome'" class="instructions" :validations="{
        SubjectID: {
          minLength: $magpie.v.minLength(2)
        }
      }">
        <!-- <WelcomeScreen /> -->
        <div style="width: 90%; margin: auto;">

        <div style="background-color: lightgrey; padding: 10px;">
          <b>Information About this study </b>
        </div>
        <p>
           Please read the text below carefully.
          <br><br>
            <b>What is investigated and how?</b> This study will help us learn about how people read. It will take you around 15 minutes to complete.
          <br><br>
            <b>Who can participate?</b> You can participate only if you are an adult native speaker of English.
          <br><br>
          <b>What am I supposed to do as a participant?</b> In this study, you will use the computer mouse to read sentences in English and answer questions about them.
          <br><br>
            <b>What are my rights during participation?</b> Your participation in this study is voluntary. If you choose to participate, you may change your mind and leave the study at any time by closing the web page.
          <br><br>
            <b>Will I be compensated?</b> If you complete the study, you will be compensated with the amount specified on prolific.co.
          <br><br>
            <b>What data is collected from me and how is it used?</b> During this study, we will track the position of your mouse on screen. The data from this study may be presented at scientific conferences and published in scientific journals, as well as in online repositories. All data will remain anonymous.
          <br><br>
            <b> What are my rights to my personal data? </b> You can request information about the personal data collected from you at any time and without giving reasons. You can also request that it be rectified, handed over to you, barred for processing or erased. To do so, please contact us via prolific messages.
        </p>

        <br>
        <div style="background-color: lightgrey; padding: 10px;">
          <b> Consent Form </b>
        </div>
        <br>
          I, the participant, confirm by clicking the button below: <br>
          <div style="padding-left: 30px"> • I have read and understood the study information. My questions have been answered completely and to my satisfaction. </div>
          <div style="padding-left: 30px">• I comply with the inclusion and exclusion criteria for participation described above. I am aware of the requirements and restrictions to be observed during the study. </div>
          <div style="padding-left: 30px">• I have had enough time to decide about my participation. </div>
          <div style="padding-left: 30px">• I participate in this study voluntarily and consent that my personal data be used as described above.</div>
          <div style="padding-left: 30px">• I understand that I can stop participating at any moment.</div>
        <br>

        <tr>
          <td>Please enter your Prolific ID or any string to continue:&nbsp</td><td><input name="ProlID" type="text" class="obligatory" v-model="$magpie.measurements.SubjectID"/></td>
        </tr>
        <tr></tr>

        </div>
          <div v-if="
            $magpie.measurements.SubjectID&&
            !$magpie.validateMeasurements.SubjectID.$invalid
            ">
          <br> By clicking on the button below you consent to participating in this study: <br><br>
          <br />
          <button 
            @click="$magpie.addExpData({ SubjectId: $magpie.measurements.SubjectID}); $magpie.nextScreen()">
            Proceed
          </button>

        </div>
     </Screen>

     <InstructionScreen :title="'Instructions'" :button-text="'Continue'">
      <div style="width: 80%; margin: auto;">
        <p></p>
        <p>In this study, you will read short texts and answer questions about them. However, unlike in normal reading, the texts will be blurred. In order to bring the text into focus move your mouse over it. Take as much time to read the text as you need in order to understand it. When you are done reading, answer the question at the bottom and click “next” to move on.</p>
        <p>Let’s get started!</p>
      </div>
    </InstructionScreen>

    <template v-for="(trial, i) of trials">
      <Screen :key="i" class="main_screen" :progress="i / trials.length">
        <Slide class="text_slide">
          <form>
            <input type="hidden" class="item_id" :value="trial.item_id">
            <input type="hidden" class="condition_id" :value="trial.condition_id">
            <input type="hidden" class="list" :value="trial.group">
            <input type="hidden" class="question_type" :value="trial.question_type">
          </form>
          <div class="oval-cursor"></div>
          <template>
            <div v-if="hasStarted" class="readingText" style= "top:5%;" @mousemove="moveCursor" @mouseleave="changeBack">
              <template v-for="(word, index) of trial.text.split(' ')">
                <span :key="index" :data-index="index" >
                  {{ word }}
                </span>
              </template>
            </div>
            <div v-if="hasStarted" class="blurry-layer" style="opacity: 0.3; filter: blur(3.5px); transition: all 0.3s linear 0s; top:5%;"> 
              {{trial.text}}
            </div>
          </template>
          <!-- Start button -->
          <button style="bottom:78%; left: -5%; transform: translate(-50%, -50%)" @click="startReading" v-if="!hasStarted">
            Start
          </button>

          <button v-if="hasStarted" style= "bottom:65%; left: 95%; transform: translate(-50%, -50%)" @click="handleRCQButton" :disabled="!hasRevealed">
            Done
          </button>
        </Slide>

        <Slide class="question_slide">
          <div class="radio-options" style="padding-top: 4%;">
            <form>
              <!-- comprehension questions and the response options -->
              <div>{{ trial.question }}</div>
              <template v-for='(option, index) of trial.response_options'>
                <label style="cursor:pointer; user-select:none; border:1px solid #ccc; border-radius:8px; padding:14px 22px; display:inline-flex; align-items:center;margin-right:12px;">
                  <input type="radio" :value="option" name="opt" v-model="$magpie.measurements.response" style="display:none;">
                  <span style="display:block;">{{ option }}</span>
                </label>
              </template>
            </form>
          </div>

          <button v-if="$magpie.measurements.response" style="transform: translate(-50%, -50%); bottom:55%" @click="$magpie.saveAndNextScreen()">
            Next
          </button>
        </Slide>
      </Screen>
    </template>

    <Screen>
      <p>1. Which input device are you using for this experiment?</p>
        <MultipleChoiceInput
            :response.sync= "$magpie.measurements.device"
            orientation="horizontal"
            :options="['Mouse', 'Trackpad', 'Stylus', 'Other']" />
      <br>
      <br>
      <p>2. What operating system did you use during this experiment?</p>
        <MultipleChoiceInput
            :response.sync="$magpie.measurements.os"
            orientation="horizontal"
            :options="['Microsoft Windows', 'Apple macOS', 'Other']" />
      <button style= "bottom: 5%; transform: translate(-50%, -50%)" @click="$magpie.saveAndNextScreen();">Next</button>
    </Screen>

    <Screen>

      <Slide>
        <p>"What unusual happened during the experiment?"</p>
        <TextareaInput
            :response.sync= "$magpie.measurements.issue"
          />
        <button @click="$magpie.saveAndNextScreen();">Submit</button>
      </Slide>

    </Screen>

    <SubmitResultsScreen />
  </Experiment>
</template>

<script>
import test from '../trials/test.tsv';
import practice from '../trials/practice.tsv';
import _ from 'lodash';

function interleaveWithFillers(items, fillers) {
  const I = [...items];  // shadow copy
  const F = [...fillers];

  if (I.length > 1 && F.length < I.length - 1) {
    throw new Error(`Need at least ${I.length - 1} fillers; got ${F.length}.`);
  }

  const separators = F.splice(0, Math.max(0, I.length - 1));
  const seq = I.length ? [I[0]] : [];

  for (let k = 0; k < separators.length; k++) {
    seq.push(separators[k], I[k + 1]);
  }

  const gaps = Array.from({ length: seq.length + 1 }, () => []);
  while (F.length) {
    gaps[(Math.random() * gaps.length) | 0].push(F.pop());
  }

  // Stitch gaps + seq
  const out = [];
  for (let i = 0; i < seq.length; i++) {
    if (gaps[i].length) out.push(...gaps[i]);
    out.push(seq[i]);
  }
  if (gaps[seq.length].length) out.push(...gaps[seq.length]);
  return out;
}

export default {
  name: 'App',
  data() {
    const trials = _.concat(practice[1], test);
    const order = ["Yes", "No"];

    const updatedTrials = trials.map(trial => ({
      ...trial,
      response_options: [trial.response_true, trial.response_distractors]
        .map(s => (s || '').replace(/ ?["]+/g, ''))
        .sort((a, b) => order.indexOf(a) - order.indexOf(b))
    }));

    return {
      hasStarted: false,
      hasRevealed: false,
      trials: updatedTrials,
      currentIndex: null,              
      showFirstDiv: true,
      mousePosition: { x: 0, y: 0 },
      _meta: null,
      _intervalId: null,                 // for cleanup
      _outsideOnceLogged: false,         // log fallback only once per outside streak
    };
  },
  mounted() {
    this._intervalId = setInterval(this.saveData, 50);
  },
  beforeDestroy() {
    if (this._intervalId) clearInterval(this._intervalId);
  },
  methods: {
    changeBack() {
      const oc = this.$el.querySelector(".oval-cursor");
      if (oc) {
        oc.classList.remove('grow');
        oc.classList.remove('blank');
      }
      this.currentIndex = null;          
      this._outsideOnceLogged = false;  
    },

    saveData() {
      if (!this.hasStarted) return; 
      const readingContainer = this.$el.querySelector('.readingText');
      if (!readingContainer) return;

      if (!this._meta) {
        const listEl = this.$el.querySelector(".list");
        const itemEl = this.$el.querySelector(".item_id");
        const condEl = this.$el.querySelector(".condition_id");
        if (!(listEl && itemEl && condEl)) return;
        this._meta = {
          List: listEl.value,
          ItemId: itemEl.value,
          Condition: condEl.value,
        };
      }

      const idx = this.currentIndex;

      if (idx !== null && idx >= 0) {
        const currentElement = readingContainer.querySelector(`span[data-index="${idx}"]`);
        if (currentElement) {
          const r = currentElement.getBoundingClientRect();
          $magpie.addTrialData({
            ...this._meta,
            Index: idx | 0,
            Word: currentElement.textContent,
            mousePositionX: this.mousePosition.x | 0,
            mousePositionY: this.mousePosition.y | 0,
            // wordPositionTop: r.top | 0,
            wordPositionLeft: r.left | 0,
            // wordPositionBottom: r.bottom | 0,
            wordPositionRight: r.right | 0
          });
          this._outsideOnceLogged = false;
        }
        return;
    }
      if (idx === -1) {
        $magpie.addTrialData({
          ...this._meta,
          Index: -1,
          mousePositionX: this.mousePosition.x | 0,
          mousePositionY: this.mousePosition.y | 0
        });
        this._outsideOnceLogged = false;
        return;
      }

      if (idx === null && !this._outsideOnceLogged) {
        $magpie.addTrialData({
          ...this._meta,
          Index: null,
          mousePositionX: this.mousePosition.x | 0,
          mousePositionY: this.mousePosition.y | 0
        });
        this._outsideOnceLogged = true;
      }
    },

    moveCursor(e) {
      if (!this.hasStarted) return;
      const oc = this.$el.querySelector(".oval-cursor");
      if (oc && !oc.classList.contains('grow')) oc.classList.add('grow');

      const x = e.clientX;
      const y = e.clientY;

      let targetSpan = e.target && e.target.closest ? e.target.closest('span') : null;

      if (!targetSpan) {
        const el = document.elementFromPoint(x, y - 10);
        targetSpan = el && el.closest ? el.closest('span') : null;
      }

      if (oc) {
        if (targetSpan) oc.classList.remove('blank'); else oc.classList.add('blank');
        oc.style.left = (x + 12) + 'px';
        oc.style.top  = (y - 6) + 'px';
      }

      if (targetSpan) {
        this.hasRevealed = true;
        this._outsideOnceLogged = false;
        this.currentIndex = targetSpan.getAttribute('data-index');
      } else {
        this.currentIndex = -1;
      }

      this.mousePosition.x = x;
      this.mousePosition.y = y;
    },

    handleRCQButton() {
      if (!this.hasRevealed) return;
      $magpie.nextSlide();
      this.hasStarted = false;
      this.hasRevealed = false;
    },
    startReading() {
      this.hasStarted = true;
      this.hasRevealed = false;
    },

    getScreenDimensions() {
      return {
        window_inner_width: window.innerWidth,
        window_inner_height: window.innerHeight
      };
    }
  },
};

</script>


<style>
  .experiment {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .main_screen {
    isolation: isolate;
    position: relative;
    width: 100%;
    height: auto;
    font-size: 18px;
    line-height: 40px;
  }
  .debugResults{
    width: 100%;
  }
  .readingText {
    /* z-index: 1; */
    position: absolute;
    color: white;
    text-align: left;
    font-weight: 450;
    cursor: pointer;
    padding-top: 2%;
    padding-bottom: 2%;
    padding-left: 0;
    padding-right: 0;
  }
  button {
    position: absolute;
    bottom: 0;
    left: 50%;
  }
  .oval-cursor {
    position: fixed;
    /* left: 10px; */
    z-index: 2;
    width: 1px;
    height: 1px;
    transform: translate(-50%, -50%);
    background-color: white;
    mix-blend-mode: difference;
    border-radius: 50%;
    pointer-events: none;
    transition: width 0.5s, height 0.5s;
  } 
  .oval-cursor.grow.blank {
    width: 80px;
    height: 38px;
  }
  .oval-cursor.grow {
    width: 102px;
    height: 38px;
    border-radius: 50%;
    box-shadow: 30px 0 8px -4px rgba(255, 255, 255, 0.1), -30px 0 8px -4px rgba(255, 255, 255, 0.1);
    background-color: rgba(255, 255, 255, 0.3);
    background-blend-mode: screen;
    pointer-events: none;
    transition: width 0.5s, height 0.5s;
    filter:blur(3px);
  }
  .oval-cursor.grow::before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 70%;
    height: 70%;
    background-color: white;
    mix-blend-mode: normal;
    border-radius: 50%;
}
  .blurry-layer {
    position: absolute;
    pointer-events: none;
    color: black;
    text-align: left;
    font-weight: 450;
    padding-top: 2%;
    padding-bottom: 2%;
    padding-left: 0;
    padding-right: 0;
  }
  * {
    user-select: none; /* Standard syntax */
    -webkit-user-select: none; /* Safari */
    -moz-user-select: none; /* Firefox */
    -ms-user-select: none; /* Internet Explorer/Edge */
    font-family: "Courier", monospace;
    }
label {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 14px 22px;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}
/* make the selected text bold + red */
label input[type="radio"]:checked + span {
  font-weight: 600;
  color: #0b3c5d;
}
/* highlight the entire selected box */
label:has(input[type="radio"]:checked) {
  box-shadow: 0 0 0 3px #0b3c5d inset;
  border-color: #0b3c5d;
}
</style>