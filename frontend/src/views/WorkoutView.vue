<template>
  <div @click="backgroundPressed()">
    <HelloHeader class="hello-header" v-bind:header="jwtData.name" v-on:click="getWorkout"/>
    <div class="todo-block">
      <div class="todo">
        <WorkoutList 
            v-bind:workouts="workouts" 
            v-on:delete-workout="deleteWorkout"
        />
        <AddWorkout/>
      </div>
    </div>
  </div>
</template>

<script>
import AddWorkout from '../components/AddWorkout'
import HelloHeader from '../components/HelloHeader.vue'
import WorkoutList from '../components/WorkoutList';
import axios from 'axios'

export default {
    name: 'Home',
        components: {
        WorkoutList,
        HelloHeader,
        AddWorkout
        },
        data() {
            return {
                user: {},
                headerItem : '',
                email : 'peter@pratgen.dk',
                password : 'safe',
                apiInstance : '',
                workouts : ''
            }
    },
    methods: {
        async init() {
            await this.login()
            this.apiInstance = this.createInstance();  
            await this.getWorkout();
        },
        async login() {
            try {
                let response = await axios.post("https://liftlog.app/api/login",
                {
                    email : this.email, 
                    password: this.password
                })
                let token = response.data
                localStorage.setItem("user", token)
            } catch (err) {
                console.log(err)
            }
        },
        createInstance() {
            const token = localStorage.getItem("user")
            return axios.create({
                baseURL: "https://liftlog.app/api",
                headers : {
                    Authorization : `Bearer ${token}`
                }
            })
        },
        async getWorkout() {
            const response = await this.apiInstance.get('/workout')
            this.workouts = JSON.parse(response.request.response)
        },
        async titleChange(data){
            let res = await this.apiInstance.post('/workout/rename',
                {
                    id : data["workoutId"],
                    title : data["title"]
                }
            )
            console.log(res)
            let ele = this.workouts.find(element => element._id == data.workoutId)
            ele.title = data.title
        },
        deleteWorkout(workoutId) {
            this.apiInstance.delete('/workout/' + workoutId )
            let ele = this.workouts.find(element => element["_id"] == workoutId)
            let index = this.workouts.indexOf(ele)
            this.workouts.splice(index, 1)
        },
        addRepetition(data){
            let workout = this.workouts.find(element => element["_id"] == data["id"])
            let exercise = workout["exerciseList"][data["index"]]
            const length = exercise["set"].length
            if (length > 0) {
                const weight = exercise["set"][length - 1]["weight"];
                const reps = exercise["set"][length - 1]["repetitions"];
                exercise["set"].push({repetitions : reps, weight : weight}) 
            } else {
                exercise["set"].push({repetitions : 0, weight : 0}) 
            }
        },
        async changeRep(data){
            let workout = this.workouts.find(element => element._id == data.workoutId)
            let exercise =  workout.exerciseList.find(element => element.id == data.exerciseId)
            let rep = exercise.set.find(element => element.id == data.repItem.id)
            rep.repetitions = data.repItem.repetitions
            rep.weight = data.repItem.weight
            let res = await this.apiInstance.put('/workout/rep_change', {
                workoutId: data["workoutId"],
                exerciseId : data["exerciseId"],
                repItem: data["repItem"]
            })
            console.log(" response " + res.data)
        },
        backgroundPressed() {
            this.emitter.emit('pressed-background')
        },
        async submitWorkout(data) {
            const res = await this.apiInstance.post('/workout', data)
            data._id =  res.data
            this.workouts.push(data)
        },
        changeExerciseName(data) {
            let workout = this.workouts.find(element => element["_id"] == data["workoutId"])
            let exercise =  workout["exerciseList"].find(element => element["id"] == data["exerciseId"])
            exercise.name = data["name"]
            this.apiInstance.put('/workout/rename_exercise', {
                id: data["workoutId"],
                exerciseId : data["exerciseId"],
                name : data["name"]
            })
        },
        deleteExercise(data) {
            let workout = this.workouts.find(element => element["_id"] == data["workoutId"])
            workout["exerciseList"].splice(data["exerciseIndex"], 1)
            this.apiInstance.post('/workout/update_exercise', {
                id: data["workoutId"],
                exerciseList : workout["exerciseList"]
            })
        },
        addExercise(id) {
            let workout = this.workouts.find(element => element["_id"] == id)
            workout["exerciseList"].push({ name: "", set: []})
        },
        deleteRep(data) {
            let workout = this.workouts.find(element => element["_id"] == data.workoutId)
            let exercise = workout.exerciseList[data.exerciseIndex]
            exercise.set.splice(data.repIndex, 1)
            this.apiInstance.post('/workout/update_exercise', {
                id: data.workoutId,
                    exerciseList : workout.exerciseList
            })
        }
    },
    computed : {
        jwtData() {
            const token = localStorage.getItem("user")
            if (token) {
                        return JSON.parse(atob(token.split('.')[1]))
            }
            else {
                return {};
            }
        }
    },
    created() {
        this.init()
    },
    mounted() {
        this.emitter.on('new-repetition', this.addRepetition)
        this.emitter.on('completed-rep-edit', this.changeRep)
        this.emitter.on('submit-new-workout', this.submitWorkout)
        this.emitter.on('exercise-name-change', this.changeExerciseName)
        this.emitter.on('delete-exercise', this.deleteExercise)
        this.emitter.on('add-item', this.addExercise)
        this.emitter.on('delete-rep', this.deleteRep)
        this.emitter.on('title-change', this.titleChange)
    }
}

</script>

<style lang="scss" >
@import "../assets/variables.scss";

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}


body {
  font-family: Arial, Helvetica, sans-serif;
  line-height: 1.4;
}
.btn {
  display: inline-block;
  border: none;
  background: #555;
  color: #fff;
  padding: 7px 20px;
  cursor: pointer;
}
.btn:hover {
  background: #666;
}
.addtodo {
  padding-top: 25px;
}

.todo {
  padding-top: 0.3rem;

}
.todo-block {
  width: $content-width;
  margin: 0 auto;
}

.hello-header {
  width: calc(#{$content-width} - 2%);
  margin: 0 auto;
}

@media only screen and (max-width: 1400px) {
  .todo-block {
    width: 60%;
  }
  .hello-header {
    width: 60%;
  }
}


@media only screen and (max-width: 1000px) {
  .todo-block {
    width: 80%;
  }
  .hello-header {
    width: 80%;
  }
}

@media only screen and (max-width: 600px) {
  .todo-block {
   width: 100%;
  }
  .hello-header {
    width: 95%;
  }
}

</style>

