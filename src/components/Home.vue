<template lang="pug">
  .content-wrapper
    section
      .container
        h1.ui-title-1 Home
        form(@submit.prevent="submit")
          .form-item(:class="{ 'error-input': $v.taskTitle.$error }")
            input(
              type="text"
              placeholder="What we will be watch?"
              v-model="taskTitle"
              :class="{ 'error': $v.taskTitle.$error }" 
              @change="$v.taskTitle.$touch()"
            )
            .error(v-if="!$v.taskTitle.required") Field is required

          .form-item(:class="{ 'error-input': $v.taskDescription.$error }")
            textarea(
              type="text"
              placeholder="Description"
              v-model="taskDescription"
              :class="{ 'error': $v.taskDescription.$error }" 
              @change="$v.taskDescription.$touch()"
            )
            .error(v-if="!$v.taskDescription.required") Field is required
          .option-list  
            input.what-watch(
              type="radio"
              id="radio-film"
              value="Film"
              v-model="whatWatch"
            )
            label(
              for="radio-film"
            ) Film

            input.what-watch(
              type="radio"
              id="radio-serial"
              value="Serial"
              v-model="whatWatch"
            )
            label(
              for="radio-serial"
            ) Serial

          .total-time
            .total-time__film(
              v-if="whatWatch === 'Film'"
            )
              span Hours 
              input.time-input(
                type="number"
                v-model="filmHours"
              )

              span Minutes
              input.time-input(
                type="number"
                v-model="filmMinutes"
              )

              p {{ filmTime }}

            .total-time__serial(
              v-if="whatWatch === 'Serial'"
            )
              span.time-title How many season?
              input.time-input(
                type="number"
                v-model="serialSeason"
              )
              span.time-title How many series?
              input.time-input(
                type="number"
                v-model="serialSeries"
              )
              span.time-title How long is one series? (minutes)
              input.time-input(
                type="number"
                v-model="serialSeriesMinutes"
              )

              p {{ serialTime }}


          // Add a new tag 
          .tag-list.tag-list--addTagUsed  
            .ui-tag__wrapper(
              @click="tagMenuShow = !tagMenuShow"
            )
              .ui-tag
                span.tag-title Add new
                span.button-close(
                  :class="{active: !tagMenuShow}"
                )

          // Show Input
          .tag-list--menu.tag-list--input(
            v-if="tagMenuShow"
          )
            input.tag-add--input(
              type="text"
              placeholder="New tag"
              v-model="tagTitle"
              @keyup.enter="newTag"
            )
            .button.button--round.button-primary(
              @click="newTag"
            ) Send

          .tag-list 
            transition-group(
              name="bounce"
              enter-active-class="bounceInRight"
              leave-active-class="bounceOutDown"
            )
              .ui-tag__wrapper(
                v-for="tag in tags"
                :key="tag.title"
              )
                .ui-tag(
                  @click="addTagUsed(tag)"
                  :class="{active: tag.use}"
                )
                  span.tag-title {{ tag.title }}
                  span.button-close

          .button-list
              button.button-primary(
                @click="newTask"
                type="submit" 
                :disabled="submitStatus === 'PENDING'"
              ) Send

          .button-list.button-list--info
            p.typo__p(v-if="submitStatus === 'OK'") Thanks for your submission!
            p.typo__p(v-if="submitStatus === 'ERROR'") Please fill the form correctly.
            p.typo__p(v-if="submitStatus === 'PENDING'") Sending...

      
</template>

<script>
import { required } from "vuelidate/lib/validators";

export default {
  data() {
    return {
      taskTitle: "",
      taskDescription: "",
      whatWatch: "Film",

      filmHours: 1,
      filmMinutes: 30,
      serialSeason: 1,
      serialSeries: 11,
      serialSeriesMinutes: 40,

      // Tags
      tagsUsed: [],
      tagMenuShow: false,
      tagTitle: "",

      submitStatus: null,
    };
  },
  validations: {
    taskTitle: {
      required,
    },
    taskDescription: {
      required,
    },
  },
  methods: {
    newTag() {
      if (this.tagTitle === "") {
        return;
      }
      const tag = {
        title: this.tagTitle,
        use: false,
      };
      this.$store.dispatch("newTag", tag);
    },
    newTask() {
      if (this.taskTitle === "") {
        return;
      }
      let time;
      if (this.whatWatch === "Film") {
        time = this.filmTime;
      } else {
        time = this.serialTime;
      }

      const task = {
        title: this.taskTitle,
        description: this.taskDescription,
        whatWatch: this.whatWatch,
        time,
        tags: this.tagsUsed,
        completed: false,
        editing: false,
      };

      this.$store.dispatch("newTask", task);

      //reset
      this.taskTitle = "";
      this.taskDescription = "";
      this.tagsUsed = [];
      this.$v.$reset();

      for (let i = 0; i < this.tags.length; i++) {
        this.tags[i].use = false;
      }
    },

    getHoursAndMinutes(minutes) {
      let hours = Math.trunc(minutes / 60);
      let min = minutes % 60;
      return hours + " Hours " + min + " Minutes";
    },

    addTagUsed(tag) {
      tag.use = !tag.use;
      if (tag.use === true) {
        this.tagsUsed.push({ title: tag.title });
      } else {
        this.tagsUsed.splice(tag.title, 1);
      }
    },

    submit() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        this.submitStatus = "ERROR";
        console.log(this.$v);
      } else {
        const movieBlock = {
          title: this.taskTitle,
          description: this.taskDescription,
        };
        console.log(movieBlock);
        this.submitStatus = "PENDING";
        setTimeout(() => {
          this.submitStatus = "OK";
        }, 500);
      }
    },
  },
  computed: {
    tags() {
      return this.$store.getters.tags;
    },
    filmTime() {
      let min = this.filmHours * 60 + this.filmMinutes;
      return this.getHoursAndMinutes(min);
    },
    serialTime() {
      let min =
        this.serialSeason * this.serialSeries * this.serialSeriesMinutes;
      return this.getHoursAndMinutes(min);
    },
  },
};
</script>

<style lang="stylus" scoped>
.option-list
 display flex
 .what-watch
   margin 0.3rem .5rem 0 0
 label
   margin-right 1.5rem
   &:last-child
     margin-right 0

.total-time
  margin-bottom 20px

.time-title
  display block
  margin-bottom 6px

.time-input
  max-width 80px
  margin-right 10px
  margin-left 10px

.tag-list
  margin-bottom 20px

.ui-tag__wrapper
  margin-right 18px
  margin-bottom 10px
  &:last-child
    margin-right 0

.ui-tag
  &.active
    background-color #444ce0
    color #fff
  .button-close
    &.active
      transform: rotate(45deg)
    &.used
      background-color #444ce0
      color #fff
      .button-close
        &:before,
        &:after
          background-color #fff

.tag-list--menu
  display flex
  justify-self space-between
  align-items center
  margin-bottom 1rem

.tag-add--input
  margin-bottom 0
  margin-right 10px
  height 42px

.form-item
  .error
    display none
  &.error-input
    .error
      display block
      color tomato
      padding-left: 10px;
      font-size: 14px;
      margin-bottom 8px

.button-list
  text-align right
  margin-bottom 20px
  &.buttons-list--info
    text-align center
    &:last-child
      margin-bottom 0
</style>
