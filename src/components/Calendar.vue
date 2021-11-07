<template>
  <v-container>
    <h3 style="text-align: center">
      To create an event, click on the date number
    </h3>
    <div id="app">
      <v-app id="inspire">
        <v-row class="fill-height">
          <v-col>
            <v-sheet height="64">
              <v-toolbar flat>
                <v-btn
                  outlined
                  class="mr-4"
                  color="grey darken-2"
                  @click="setToday"
                >
                  Today
                </v-btn>
                <v-btn fab text small color="grey darken-2" @click="prev">
                  <v-icon small> mdi-chevron-left </v-icon>
                </v-btn>
                <v-btn fab text small color="grey darken-2" @click="next">
                  <v-icon small> mdi-chevron-right </v-icon>
                </v-btn>
                <v-toolbar-title v-if="$refs.calendar">
                  {{ $refs.calendar.title }}
                </v-toolbar-title>
                <v-spacer></v-spacer>
                <v-menu bottom right>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      outlined
                      color="grey darken-2"
                      v-bind="attrs"
                      v-on="on"
                    >
                      <span>{{ typeToLabel[type] }}</span>
                      <v-icon right> mdi-menu-down </v-icon>
                    </v-btn>
                  </template>
                  <v-list>
                    <v-list-item @click="type = 'day'">
                      <v-list-item-title>Day</v-list-item-title>
                    </v-list-item>
                    <v-list-item @click="type = 'week'">
                      <v-list-item-title>Week</v-list-item-title>
                    </v-list-item>
                    <v-list-item @click="type = 'month'">
                      <v-list-item-title>Month</v-list-item-title>
                    </v-list-item>
                    <v-list-item @click="type = '4day'">
                      <v-list-item-title>4 days</v-list-item-title>
                    </v-list-item>
                  </v-list>
                </v-menu>
              </v-toolbar>
            </v-sheet>

            <v-dialog v-model="dialogDate" max-width="500" @click:outside = "resetForm" >
              <v-card>
                <v-container>
                  <v-form @submit.prevent="addEvent">
                    <v-text-field
                      v-model="name"
                      type="text"
                      label="event name (required)"
                    ></v-text-field>
                    <v-text-field
                      v-model="description"
                      type="text"
                      label="description"
                    ></v-text-field>
                    <v-text-field
                      v-model="start"
                      type="date"
                      label="start (required)"
                    ></v-text-field>
                    <v-text-field
                      v-model="end"
                      type="date"
                      label="end (required)"
                    ></v-text-field>
                    <v-text-field
                      v-model="color"
                      type="color"
                      label="color (click to open color menu)"
                    ></v-text-field>
                    <v-btn
                      type="submit"
                      color="primary"
                      class="mr-4"
                      @click.stop="dialog = false"
                    >
                      create event
                    </v-btn>
                  </v-form>
                </v-container>
              </v-card>
            </v-dialog>
            <v-sheet height="600">
              <v-calendar
                ref="calendar"
                v-model="focus"
                color="primary"
                :events="events"
                :event-color="getEventColor"
                :type="type"
                @click:event="showEvent"
                @click:more="viewDay"
                @click:date="setDialogDate"
                @change="updateRange"
              ></v-calendar>
              <v-menu
                v-model="selectedOpen"
                :close-on-content-click="false"
                :activator="selectedElement"
                offset-x
              >
                <v-card color="grey lighten-4" min-width="350px" flat>
                  <v-toolbar :color="selectedEvent.color" dark>
                    <v-btn v-show="!updateMode" @click="updateMode = true" icon>
                      <v-icon>mdi-pencil</v-icon>
                    </v-btn>
                    <v-btn v-show="updateMode" @click="editEvent" icon>
                      <v-icon>mdi-check</v-icon>
                    </v-btn>
                    <v-btn v-show="updateMode" @click="updateMode = false" icon>
                      X
                    </v-btn>
                    <v-toolbar-title
                      v-show="!updateMode"
                      v-html="selectedEvent.name"
                    ></v-toolbar-title>
                    <v-toolbar-title>
                      <v-text-field v-show="updateMode" v-model="newName">
                      </v-text-field>
                    </v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn icon @click="deleteEvent(selectedEvent._id)">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>
                  </v-toolbar>
                  <v-card-text>
                    <span
                      v-show="!updateMode"
                      v-html="selectedEvent.description"
                    ></span>
                    <v-text-field v-show="updateMode" v-model="newDescription">
                    </v-text-field>
                    <br />
                    <span v-show="isHeDate">{{ heDate }}</span>
                  </v-card-text>
                  <v-card-actions>
                    <v-btn text color="secondary" @click="changeDadeToHe">{{
                      isHeDate ? "Close hebrew date" : "Show date in hebrew"
                    }}</v-btn>
                    <v-btn text color="secondary" @click="selectedOpen = false">
                      Cancel
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-menu>
            </v-sheet>
          </v-col>
        </v-row>
      </v-app>
    </div>
  </v-container>
</template>

<script>
import axios from "axios";
import Hebcal from "hebcal";

export default {
  name: "Calendar",
  data: () => ({
    focus: "",
    type: "month",
    typeToLabel: {
      month: "Month",
      week: "Week",
      day: "Day",
      "4day": "4 Days",
    },
    name: null,
    description: null,
    start: null,
    end: null,
    color: "#1976D2", //Defoult color,
    selectedEvent: {},
    selectedElement: null,
    selectedOpen: false,
    events: [],
    updateMode: false,
    dialog: false,
    dialogDate: false,
    newName: null,
    newDescription: null,
    heDate: null,
    isHeDate: false,
  }),
  mounted() {
    this.$refs.calendar.checkChange();
  },
  methods: {
    viewDay({ date }) {
      this.focus = date;
      this.type = "day";
    },
    setDialogDate({ date }) {
      this.dialogDate = true;
      this.focus = date;
      this.start = date;
      this.end = date;
    },
    getEventColor(event) {
      return event.color;
    },
    setToday() {
      this.focus = "";
    },
    prev() {
      this.$refs.calendar.prev();
    },
    next() {
      this.$refs.calendar.next();
    },
    showEvent({ nativeEvent, event }) {
      const open = () => {
        this.selectedEvent = event;
        this.newName = event.name;
        this.newDescription = event.description;
        this.selectedElement = nativeEvent.target;

        requestAnimationFrame(() =>
          requestAnimationFrame(() => (this.selectedOpen = true))
        );
      };

      if (this.selectedOpen) {
        this.selectedOpen = false;
        this.isHeDate = false;
        requestAnimationFrame(() => requestAnimationFrame(() => open()));
      } else {
        open();
      }
      nativeEvent.stopPropagation();
    },

    updateRange() {
      const events = [];
      axios.get("/api/event").then((res) => {
        this.events = res.data;
      });
      this.events = events;
    },

    async editEvent() {
      const res = await axios.put("/api/event", {
        _id: this.selectedEvent._id,
        name: this.newName,
        description: this.newDescription,
      });
      const index = this.events.findIndex((e) => e._id == res.data._id);
      this.events[index].name = this.newName;
      this.events[index].description = this.newDescription;
      this.updateMode = false;
    },

    deleteEvent(id) {
      var result = confirm("Want to delete this event ?");
      if (result) {
        axios.delete(`/api/event/${id}`).then((response) => {
          console.log(response.data);
        });
      }
      this.selectedOpen = false;
      this.events = this.events.filter((e) => e._id !== id);
    },

    resetForm() {
      this.name = "",
      this.description = "";
      this.start = "";
      this.end = "";
      this.color = "";
    },

    addEvent() {
      const eventObj = {
        name: this.name,
        description: this.description,
        start: this.start,
        end: this.end,
        color: this.color,
      };
      if (this.name && this.start && this.end) {
          axios.post("/api/event", eventObj).then((response) => {
          this.selectedEvent._id = response.data.id,
          this.dialogDate = false;
          this.resetForm();
          this.events.push(response.data);
        });
      } else {
        alert("You must enter event name, start, and end time");
      }
    },

    changeDadeToHe() {
      let start = new Date(this.selectedEvent.start);
      let end = new Date(this.selectedEvent.end);
      const heStarsDate = new Hebcal.HDate(start).toString("h");
      const heEndDate = new Hebcal.HDate(end).toString("h");
      if (heStarsDate == heEndDate) {
        this.heDate = heStarsDate;
      } else {
        this.heDate = heStarsDate + "  -  " + heEndDate;
      }
      this.isHeDate = !this.isHeDate;
    },
  },
};
</script>
