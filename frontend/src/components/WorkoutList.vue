<template>
 <div class="list">
  <div v-bind:key="workout.dateCreated" v-for="workout in workouts" >
    <Workout 
        v-bind:workout="workout" 
        v-on:delete-workout="emitDeleteWorkout"
    />
  </div>
 </div> 
</template>

<script>
import Workout from './Workout.vue'

export default {
  name: "WorkoutList", 
  components: {
    Workout
  },
  props: ["workouts"],
  emit: ['title-change', 'delete-workout'],
  created() {
  },
    methods: {
        emitTitleChange(id, name) {
            this.$emit('title-change', id, name)            
        },
        emitDeleteWorkout(id){
            this.$emit('delete-workout', id)
        },
    }
}
</script>

<style scoped>


@media only screen and (max-width: 600px) {
}

</style>
