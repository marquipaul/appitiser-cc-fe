<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="10" md="10">
      <v-card>
        <v-card-title class="headline">
          Appitiser Code Challenge
        </v-card-title>
        <v-card-text>
          <v-form>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  label="Event Name"
                  v-model="form.name"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-menu
                  v-model="menu1"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="form.start"
                      label="Pick Start Date"
                      prepend-icon="mdi-calendar"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="form.start"
                    @input="menu1 = false"
                  ></v-date-picker>
                </v-menu>
              </v-col>
              <v-col cols="6">
                <v-menu
                  v-model="menu2"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="form.end"
                      label="Pick Start Date"
                      prepend-icon="mdi-calendar"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="form.end"
                    @input="menu2 = false"
                  ></v-date-picker>
                </v-menu>
              </v-col>
              <v-col cols="12" class="d-flex justify-center">
                <v-checkbox
                  v-for="(day, i) in days"
                  :key="i"
                  class="px-5"
                  v-model="form.days"
                  :label="day"
                  :value="day"
                ></v-checkbox>
              </v-col>
              <v-col cols="12">
                <v-btn block class="primary" @click="save()" :loading="loading">Save</v-btn>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-text class="d-flex">
          <v-btn
            icon
            class="ma-2"
            @click="$refs.calendar.prev()"
          >
            <v-icon>mdi-chevron-left</v-icon>
          </v-btn>
          <v-select
            v-model="type"
            :items="types"
            dense
            outlined
            hide-details
            class="ma-2"
            label="type"
          ></v-select>
          <v-select
            v-model="mode"
            :items="modes"
            dense
            outlined
            hide-details
            label="event-overlap-mode"
            class="ma-2"
          ></v-select>
          <v-select
            v-model="weekday"
            :items="weekdays"
            dense
            outlined
            hide-details
            label="weekdays"
            class="ma-2"
          ></v-select>
          <v-spacer></v-spacer>
          <v-btn
            icon
            class="ma-2"
            @click="$refs.calendar.next()"
          >
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
        </v-card-text>
        <v-card-text>
          <v-calendar
            ref="calendar"
            v-model="value"
            :weekdays="weekday"
            :type="type"
            :events="events"
            :event-overlap-mode="mode"
            :event-overlap-threshold="30"
          ></v-calendar>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>
<script>
export default {
  data () {
    return {
      loading: false,
      menu1: false,
      menu2: false,
      form: {
        dates: [],
        days: []
      },
      type: 'month',
      types: ['month', 'week', 'day', '4day'],
      mode: 'stack',
      modes: ['stack', 'column'],
      weekday: [0, 1, 2, 3, 4, 5, 6],
      weekdays: [
        { text: 'Sun - Sat', value: [0, 1, 2, 3, 4, 5, 6] },
        { text: 'Mon - Sun', value: [1, 2, 3, 4, 5, 6, 0] },
        { text: 'Mon - Fri', value: [1, 2, 3, 4, 5] },
        { text: 'Mon, Wed, Fri', value: [1, 3, 5] },
      ],
      days: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'],
      value: '',
      events: [],
    }
  },
  // computed: {
  //   dateRangeText () {
  //     return this.form.dates.join(' ~ ')
  //   },
  // },
  mounted () {
    this.getEvent()
  },
  methods: {
    save () {
      this.loading = true
      if (this.form.id) {
        this.$axios.put(`http://127.0.0.1:8000/api/events/${this.form.id}`, this.form)
        .then(res => {
          this.getEvent()
          this.loading = false
        })
        .catch(error => {
          console.log(error)
          this.loading = false
        })
      } else {
        this.$axios.post(`http://127.0.0.1:8000/api/events`, this.form)
          .then(res => {
            this.getEvent()
            this.loading = false
          })
          .catch(error => {
            console.log(error)
            this.loading = false
          })
      }
    },
    getEvent () {
      this.events = []
      this.loading = true
      this.$axios.get('http://127.0.0.1:8000/api/events')
        .then(res => {
          this.form = res.data
          this.form.dates = []
          this.form.dates.push(res.data.start)
          this.form.dates.push(res.data.end)
          this.form.days = JSON.parse(res.data.days)
          console.log(this.form)
          this.enumerateDaysBetweenDates(this.form.start, this.form.end)
          this.loading = false
        })
        .catch(error => {
          console.log(error)
        })
    },
    enumerateDaysBetweenDates(startDate, endDate) {
      let currDate = this.$moment(startDate).startOf('day');
      let lastDate = this.$moment(endDate).startOf('day');

      // add the first and last date if day includes to the lis of days
      if (this.form.days.includes(currDate.format('dddd'))) {
        this.events.push({
          name: this.form.name,
          start: currDate.clone().format('YYYY-MM-DD'),
        })
      }

      if (this.form.days.includes(lastDate.format('dddd'))) {
        this.events.push({
          name: this.form.name,
          start: lastDate.clone().format('YYYY-MM-DD'),
        })
      }

      while(currDate.add(1, 'days').diff(lastDate) < 0) {
          // add the event date if day includes to the lis of days
          if (this.form.days.includes(currDate.format('dddd'))) {
            this.events.push({
              name: this.form.name,
              start: currDate.clone().format('YYYY-MM-DD'),
            })
          }
      }

      console.log(this.events)
    }
  }
}
</script>
