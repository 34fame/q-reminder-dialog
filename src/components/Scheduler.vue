<template>
   <q-dialog v-model="showScheduler" persistent>
      <q-card class="q-pa-md" style="width: 600px; max-width: 95%;">
         <div class="column items-center q-gutter-md full-width">
            <q-form
               autocorrect="off"
               autocapitalize="off"
               autocomplete="off"
               spellcheck="false"
               class="full-width"
               ref="form"
            >
               <div class="row justify-between q-gutter-xs">
                  <!-- Reminder -->
                  <div class="col-6">
                     <div class="text-subtitle1">Reminder</div>
                     <q-input
                        dense
                        filled
                        hide-bottom-space
                        hide-hint
                        hint="Take aspirin, Dr appointment, Go to bed"
                        :rules="[val => !!val || 'Field is required']"
                        v-model="formData.reminder"
                     />
                  </div>

                  <!-- For -->
                  <div class="col-5">
                     <div class="text-subtitle1">For</div>
                     <q-item class="bg-grey-3 rounded-borders" clickable v-ripple style="min-height: 40px;">
                        <q-item-section>
                           <div class="row items-center q-gutter-sm">
                              <q-avatar size="24px">
                                 <q-img :src="formHelpers.user ? formHelpers.user.photo : ''"/>
                              </q-avatar>
                              <div class="text-body2">{{ formHelpers.user ? formHelpers.user.label : '' }}</div>
                           </div>
                        </q-item-section>
                        <q-menu>
                           <q-list>
                              <q-item
                                 clickable
                                 dense
                                 :key="u.id"
                                 v-close-popup
                                 v-for="u in options.users"
                                 v-ripple
                                 @click="formHelpers.user = u"
                              >
                                 <q-item-section avatar>
                                    <q-avatar>
                                       <q-img :src="u.photo"/>
                                    </q-avatar>
                                 </q-item-section>
                                 <q-item-section>
                                    <q-item-label>{{ u.label }}</q-item-label>
                                 </q-item-section>
                              </q-item>
                           </q-list>
                        </q-menu>
                     </q-item>
                  </div>
               </div>

               <!-- Related To... -->
               <div class="q-mt-md row q-gutter-xs">
                  <div class="col-12">
                     <div class="text-subtitle1">Related To...</div>
                     <div class="q-mt-xs q-gutter-sm">
                        <q-radio dense v-model="formHelpers.collection" val="medicines" label="Medicine"/>
                        <q-radio dense v-model="formHelpers.collection" val="doctors" label="Doctor"/>
                        <q-radio dense v-model="formHelpers.collection" val="trackers" label="Trackers"/>
                        <q-radio dense v-model="formHelpers.collection" val="users" label="User"/>
                     </div>
                     <div class="q-mt-md full-width">
                        <q-input
                           borderless
                           dense
                           filled
                           :rules="[val => !!val || 'Field is required']"
                           v-model="formHelpers.linked.label"
                        >
                           <q-menu>
                              <q-list>
                                 <q-item
                                    clickable
                                    :key="u.id"
                                    v-close-popup
                                    v-for="u in options[formHelpers.collection]"
                                    v-ripple
                                    @click="formHelpers.linked = u"
                                 >
                                    <template v-if="u.photo">
                                       <q-item-section avatar>
                                          <q-avatar>
                                             <q-img :src="u.photo"/>
                                          </q-avatar>
                                       </q-item-section>
                                    </template>
                                    <q-item-section>
                                       <q-item-label>{{ u.label }}</q-item-label>
                                    </q-item-section>
                                 </q-item>
                              </q-list>
                           </q-menu>
                        </q-input>
                     </div>
                  </div>
               </div>

               <!-- Starting -->
               <div class="row q-gutter-xs full-width">
                  <div class="col-12 q-gutter-sm">
                     <div class="text-subtitle1">Starting</div>
                     <div class="row q-gutter-sm wrap">
                        <q-btn
                           :color="formHelpers.starting === 'today' ? 'grey-8' : 'grey-5'"
                           :outline="formHelpers.starting !=='today'"
                           rounded
                           label="today"
                           no-caps
                           @click="onClickStarting('today')"
                        />
                        <q-btn
                           :color="formHelpers.starting === 'tomorrow' ? 'grey-8' : 'grey-5'"
                           :outline="formHelpers.starting !=='tomorrow'"
                           rounded
                           label="tomorrow"
                           no-caps
                           @click="onClickStarting('tomorrow')"
                        />
                        <q-btn
                           :color="formHelpers.starting === 'custom' ? 'grey-8' : 'grey-5'"
                           :outline="formHelpers.starting !=='custom'"
                           rounded
                           label="pick another day"
                           no-caps
                        >
                           <q-popup-proxy ref="dateProxy" transition-show="scale" transition-hide="scale">
                              <q-date
                                 color="grey-8"
                                 mask="YYYYMMDD"
                                 minimal
                                 :options="displayDateOptions"
                                 v-model="formHelpers.date"
                                 @input="(v) => onDateInput(v)"
                              >
                                 <template v-slot:default>
                                    <div class="row justify-end">
                                       <q-btn color="primary" label="Done" no-caps @click="$refs.dateProxy.hide()"/>
                                    </div>
                                 </template>
                              </q-date>
                           </q-popup-proxy>
                        </q-btn>
                        <q-btn
                           :color="formHelpers.starting === 'custom' ? 'grey-8' : 'grey-5'"
                           :outline="formHelpers.starting !=='custom'"
                           rounded
                           label="pick another time"
                           no-caps
                        >
                           <q-popup-proxy ref="timeProxy" transition-show="scale" transition-hide="scale">
                              <q-time
                                 color="grey-8"
                                 mask="HHmm"
                                 v-model="formHelpers.time"
                              >
                                 <template v-slot:default>
                                    <div class="row justify-end">
                                       <q-btn color="primary" label="Done" no-caps
                                              @click="$refs.timeProxy.hide()"/>
                                    </div>
                                 </template>
                              </q-time>
                           </q-popup-proxy>
                        </q-btn>
                     </div>

                     <div class="row justify-between items-center q-gutter-sm">
                        <!-- Date -->
                        <div class="col-6 q-gutter-xs">
                           <div class="text-subtitle2">Date</div>
                           <q-input
                              borderless
                              dense
                              filled
                              readonly
                              :value="displayDate()"
                           />
                        </div>

                        <!-- Time -->
                        <div class="col-5 q-gutter-xs">
                           <div class="text-subtitle2">Time</div>
                           <q-input
                              borderless
                              dense
                              filled
                              readonly
                              :value="displayTime()"
                           />
                        </div>
                     </div>
                  </div>
               </div>

               <!-- Recurrence Expander -->
               <div class="q-mt-lg row q-gutter-xs">
                  <div class="q-mt-sm col-12">
                     <div class="row items-center q-gutter-sm">
                        <q-btn
                           color="primary"
                           dense
                           flat
                           :icon="recurrence_expanded ? 'arrow_drop_down' : 'arrow_right'"
                           round
                           size="sm"
                           @click="recurrence_expanded = !recurrence_expanded"
                        />
                        <div class="text-caption text-primary">Recurrence</div>
                     </div>
                     <q-separator color="primary"/>
                  </div>
               </div>

               <template v-if="recurrence_expanded">
                  <transition-group
                     enter-active-class="animated fadeIn"
                     leave-active-class="animated fadeOut"
                  >
                     <!-- Recurrence -->
                     <div class="row q-mt-md q-px-xs q-gutter-xs" key="recurrence">
                        <div class="col-12 q-gutter-sm full-width">
                           <div class="row q-gutter-xs full-width">
                              <div class="col-12">
                                 <div class="text-subtitle1">Recurrence</div>
                              </div>
                              <div class="col-12">
                                 <div class="row q-gutter-sm wrap">
                                    <q-btn
                                       :color="formHelpers.recurrence === 'once' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='once'"
                                       rounded
                                       label="once"
                                       no-caps
                                       @click="onClickRecurrence('once')"
                                    />
                                    <q-btn
                                       :color="formHelpers.recurrence === 'minutely' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='minutely'"
                                       rounded
                                       label="minutely"
                                       no-caps
                                       @click="onClickRecurrence('minutely')"
                                    />
                                    <q-btn
                                       :color="formHelpers.recurrence === 'hourly' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='hourly'"
                                       rounded
                                       label="hourly"
                                       no-caps
                                       @click="onClickRecurrence('hourly')"
                                    />
                                    <q-btn
                                       :color="formHelpers.recurrence === 'daily' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='daily'"
                                       rounded
                                       label="daily"
                                       no-caps
                                       @click="onClickRecurrence('daily')"
                                    />
                                    <q-btn
                                       :color="formHelpers.recurrence === 'weekly' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='weekly'"
                                       rounded
                                       label="weekly"
                                       no-caps
                                       @click="onClickRecurrence('weekly')"
                                    />
                                    <q-btn
                                       :color="formHelpers.recurrence === 'monthly' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='monthly'"
                                       rounded
                                       label="monthly"
                                       no-caps
                                       @click="onClickRecurrence('monthly')"
                                    />
                                    <q-btn
                                       :color="formHelpers.recurrence === 'annually' ? 'grey-8' : 'grey-5'"
                                       :outline="formHelpers.recurrence !=='annually'"
                                       rounded
                                       label="annually"
                                       no-caps
                                       @click="onClickRecurrence('annually')"
                                    />
                                    <template v-if="formHelpers.recurrence !== 'once'">
                                       <q-btn
                                          color="accent"
                                          :outline="!formHelpers.recurrenceCustom"
                                          rounded
                                          :label="formHelpers.recurrenceCustom ? 'hide options' : 'show options'"
                                          no-caps
                                          @click="formHelpers.recurrenceCustom = !formHelpers.recurrenceCustom"
                                       />
                                    </template>
                                 </div>
                              </div>
                           </div>
                        </div>
                     </div>

                     <template v-if="this.formHelpers.recurrenceCustom">
                        <transition-group
                           enter-active-class="animated fadeIn"
                           leave-active-class="animated fadeOut"
                           key="trans"
                        >
                           <div class="q-mt-md row q-gutter-xs" key="options">
                              <div class="q-mt-xs col-12 q-gutter-xs">
                                 <div class="row justify-between q-getter-xs full-width">
                                    <!-- Interval -->
                                    <div class="col-6 q-gutter-xs" style="padding-left: 16px;">
                                       <div class="text-subtitle1">Repeat Every</div>
                                       <div class="row items-end q-gutter-sm">
                                          <div class="col-7">
                                             <q-input
                                                dense
                                                filled
                                                hide-bottom-space
                                                hide-hint
                                                hint="Repeat every how many hours, days, weeks?"
                                                key="intervalInput"
                                                :rules="[val => val > 0 || 'Must be positive']"
                                                type="number"
                                                v-model="formHelpers.interval"
                                             />
                                          </div>
                                          <div class="col-4 text-caption">{{ recurrenceText }}...</div>
                                       </div>
                                    </div>

                                    <!-- Set Position -->
                                    <div class="col-5 q-gutter-xs">
                                       <div class="text-subtitle1">Repeat at position</div>
                                       <q-input
                                          dense
                                          :disable="false"
                                          filled
                                          hide-bottom-space
                                          hide-hint
                                          hint="Set recurrence to Monthly and Only on weekday to Wednesday.  Now set position to 4.  Now you have 4th Wednesday of each month."
                                          key="positionInput"
                                          :rules="[val => (val >= -366 && val <= 366) || 'Must be between -1 and -366 or 1 and 366']"
                                          type="number"
                                          v-model="formHelpers.bySetPos"
                                       />
                                    </div>
                                 </div>

                                 <div class="row justify-between q-gutter-xs full-width">
                                    <!-- By Week Day -->
                                    <div class="col-6 q-gutter-xs">
                                       <div class="text-subtitle1">On these days</div>
                                       <q-select
                                          dense
                                          filled
                                          key="weekDaysSelect"
                                          multiple
                                          :options="options.daysOfWeek"
                                          v-model="formHelpers.byDay"
                                       />
                                    </div>

                                    <!-- By Month -->
                                    <div class="col-5 q-gutter-xs">
                                       <div class="text-subtitle1">On these months</div>
                                       <q-select
                                          dense
                                          filled
                                          key="monthsSelect"
                                          multiple
                                          :options="options.monthsOfYear"
                                          v-model="formHelpers.byMonth"
                                       />
                                    </div>
                                 </div>

                                 <div class="row justify-between q-gutter-xs full-width">
                                    <!-- By Month Days -->
                                    <div class="col-6 q-gutter-xs">
                                       <div class="text-subtitle1">On these days of month</div>
                                       <q-input
                                          dense
                                          filled
                                          hide-bottom-space
                                          hide-hint
                                          hint="Enter '1,5,19' for 1st, 5th and 19th of every month"
                                          key="monthDaysInput"
                                          :rules="[val => val.match(/^(?:[1-9]|[1-2][0-9]|3[0-1])?(,{1}(?:[1-9]|[1-2][0-9]|3[0-1])?)*$/)   || 'Must be numbers between 1 and 31 separated by commas']"
                                          v-model="formHelpers.byMonthDay"
                                       />
                                    </div>
                                    <!-- By Year Days -->
                                    <div class="col-5 q-gutter-xs">
                                       <div class="text-subtitle1">On these days of year</div>
                                       <q-input
                                          dense
                                          filled
                                          hide-bottom-space
                                          hide-hint
                                          hint="Enter '1,5,200' for 1st, 5th and 200th day of every year"
                                          key="yearDaysInput"
                                          :rules="[val => val.match(/^(?:[1-9]|[1-9][0-9]|[1-2][0-9][0-9]|3[0-5][0-9]|36[0-5])?(,{1}(?:[1-9]|[1-9][0-9]|[1-2][0-9][0-9]|3[0-5][0-9]|36[0-5]){1})*$/)   || 'Must be numbers between 1 and 365 separated by commas']"
                                          v-model="formHelpers.byYearDay"
                                       />
                                    </div>
                                 </div>

                                 <div class="row justify-between q-gutter-xs full-width">
                                    <!-- By Hour -->
                                    <div class="col-6 q-gutter-xs">
                                       <div class="text-subtitle1">On these hour intervals</div>
                                       <q-input
                                          dense
                                          filled
                                          hide-bottom-space
                                          hide-hint
                                          hint="Enter '4,10,18' for hours of 4am, 10am and 6pm"
                                          key="byHoursInput"
                                          :rules="[val => val.match(/^(?:[0-9]|1[0-9]|2[0-3])?(,{1}(?:[0-9]|1[0-9]|2[0-3]){1})*$/) || 'Must be numbers between 0 and 23 separated by commas']"
                                          v-model="formHelpers.byHour"
                                       />
                                    </div>
                                    <div class="col-5 q-gutter-xs">
                                       <div class="text-subtitle1">On these minute intervals</div>
                                       <q-input
                                          dense
                                          filled
                                          hide-bottom-space
                                          hide-hint
                                          hint="Enter '0,15,30,45' for top, 15th, 30th and 45th minute of the hour"
                                          key="byMinutesInput"
                                          :rules="[val => val.match(/^(?:[0-9]|[1-5][0-9]|60)?(,{1}(?:[0-9]|[1-5][0-9]|60){1})*$/) || 'Must be numbers between 0 and 59 separated by commas']"
                                          v-model="formHelpers.byMinute"
                                       />
                                    </div>
                                 </div>

                                 <div class="row justify-between q-gutter-xs full-width">
                                    <!-- Week Start -->
                                    <div class="col-6 q-gutter-xs">
                                       <div class="text-subtitle1">Week starts on</div>
                                       <div class="full-width">
                                          <q-select
                                             dense
                                             filled
                                             key="weekStarsSelect"
                                             :options="options.daysOfWeek"
                                             v-model="formHelpers.wkst"
                                          />
                                       </div>
                                    </div>

                                    <div class="col-5 q-gutter-xs">
                                    </div>
                                 </div>

                              </div>
                           </div>
                        </transition-group>
                     </template>

                     <template v-if="this.formHelpers.recurrence !== 'once'">
                        <div class="q-px-xs row q-gutter-xs wrap" key="upcoming">
                           <!-- Upcoming Reminders -->
                           <div class="q-mt-lg col-12 q-gutter-xs">
                              <div class="text-subtitle1">Upcoming Reminders</div>
                              <q-list bordered separator>
                                 <q-item v-for="timestamp in rruleSample" :key="timestamp">
                                    <q-item-section>
                                       <q-item-label class="text-grey-7 text-italic">{{
                                             timestamp
                                          }}
                                       </q-item-label>
                                    </q-item-section>
                                 </q-item>
                              </q-list>
                           </div>
                        </div>
                     </template>
                  </transition-group>
               </template>

               <!-- Optional Expander -->
               <div class="row q-mt-lg q-gutter-xs">
                  <div class="q-mt-sm col-12">
                     <div class="row items-center q-gutter-sm">
                        <q-btn
                           color="primary"
                           dense
                           flat
                           :icon="optional_expanded ? 'arrow_drop_down' : 'arrow_right'"
                           round
                           size="sm"
                           @click="optional_expanded = !optional_expanded"
                        />
                        <div class="text-caption text-primary">Optional</div>
                     </div>
                     <q-separator color="primary"/>
                  </div>
               </div>

               <transition-group
                  appear
                  enter-active-class="animated fadeIn"
                  leave-active-class="animated fadeOut"
               >
                  <template v-if="optional_expanded">

                     <!-- Send Reminders To... -->
                     <div class="q-mt-md row q-gutter-xs" key="sendReminders">
                        <div class="col-12 q-gutter-xs">
                           <div class="text-subtitle1">Send Reminders To...</div>
                           <div class="q-mt-xs q-gutter-sm">
                              <q-toggle disable label="In-App Reminder" v-model="formData.sendTo.inApp"/>
                              <q-toggle
                                 :disable="!formHelpers.user.mobile"
                                 label="Mobile (SMS)"
                                 v-model="formData.sendTo.mobile"
                              />
                              <q-toggle
                                 :disable="!formHelpers.user.email"
                                 label="Email"
                                 v-model="formData.sendTo.email"
                              />
                           </div>
                        </div>
                     </div>

                     <!-- Notes -->
                     <div class="row q-gutter-xs" key="notes">
                        <div class="col-12 q-gutter-xs">
                           <div class="text-subtitle1">Notes</div>
                           <q-editor v-model="formData.notes" min-height="5rem"/>
                        </div>
                     </div>
                  </template>
               </transition-group>

               <!-- Buttons -->
               <div class="row q-mt-lg justify-end q-gutter-md wrap">
                  <q-btn outline label="Cancel" no-caps @click="onCancel()"/>
                  <q-btn color="primary" label="Save" no-caps @click="() => onSubmit()"/>
               </div>
            </q-form>
         </div>
      </q-card>
   </q-dialog>
</template>

<script>
import moment from 'moment'
import { rrulestr } from 'rrule'

export default {
   props: {
      showScheduler: {
         type: Boolean,
         default: false,
      },
      closeScheduler: {
         type: Function,
         required: true,
      },
      handleSubmit: {
         type: Function,
         required: true,
      },
   },

   data() {
      return {
         formData: {
            notes: '',
            reminder: '',
            userId: '',
            sendTo: {
               inApp: true,
               email: false,
               mobile: false,
            },
            rrule: '',
            linked: {},
         },
         formHelpers: {
            date: '',
            time: '',
            starting: '',
            recurrence: '',
            recurrenceCustom: false,
            interval: 1,
            bySetPos: 0,
            wkst: { value: 'SU', label: 'Sunday' },
            byDay: [],
            byMonth: [],
            byMonthDay: '',
            byYearDay: '',
            byHour: '',
            byMinute: '',
            collection: 'medicines',
            linked: [],
            user: {},
         },
         recurrenceText: '',
         rrule: {
            DTSTART: {
               TZID: 'America/Chicago',
               DATE: '',
               TIME: '',
               result: '',
            },
            RRULE: {
               FREQ: 'DAILY',
               COUNT: 1,
               INTERVAL: 1,
               WKST: 'SU',
               BYDAY: null,
               BYMONTH: null,
               BYSETPOS: 0,
               BYMONTHDAY: null,
               BYYEARDAY: null,
               BYHOUR: null,
               BYMINUTE: null,
               BYSECOND: 0,
               result: '',
            },
         },
         rruleString: '',
         rruleText: '',
         rruleSample: '',
         options: {
            daysOfWeek: [
               { label: 'Sunday', value: 'SU' },
               { label: 'Monday', value: 'MO' },
               { label: 'Tuesday', value: 'TU' },
               { label: 'Wednesday', value: 'WE' },
               { label: 'Thursday', value: 'TH' },
               { label: 'Friday', value: 'FR' },
               { label: 'Saturday', value: 'SA' },
            ],
            monthsOfYear: [
               { label: 'January', value: '1' },
               { label: 'February', value: '2' },
               { label: 'March', value: '3' },
               { label: 'April', value: '4' },
               { label: 'May', value: '5' },
               { label: 'June', value: '6' },
               { label: 'July', value: '7' },
               { label: 'August', value: '8' },
               { label: 'September', value: '9' },
               { label: 'October', value: '10' },
               { label: 'November', value: '11' },
               { label: 'December', value: '12' },
            ],
            medicines: [
               { value: 'zoloft', label: 'Zoloft', subtitle: 'Sertraline 100 MG Oral Tablet' },
               { value: 'armour', label: 'Armour Thyroid', subtitle: 'thyroid (USP) 120 MG Oral Tablet' },
               { value: 'vyvanse', label: 'Vyvanse', subtitle: 'lisdexamfetamine dimesylate 30 MG Oral Capsule' },
            ],
            doctors: [
               { value: 'carrie', label: 'Carrie Blades, MD', subtitle: 'Physician/Emergency Medicine' },
               { value: 'manish', label: 'Manish Parikh, MD', subtitle: 'Physician/Internal Medicine' },
               { value: 'james', label: 'James Toung, MD', subtitle: 'Physician/Otolaryngology' },
            ],
            trackers: [
               { value: 'tracker-weight', label: 'Weight Tracker' },
            ],
            users: [
               {
                  value: 'troymoreland',
                  photo: 'https://lh3.googleusercontent.com/a-/AOh14GjAVUevGVUjVP90JxIgjpYTTz0JbJcYP0UBXdxIGQ',
                  label: 'Troy Moreland',
                  email: 'troy@morelands.net',
                  mobile: '+17139623345',
               },
               {
                  value: 'daniellemoreland',
                  photo: 'https://lh3.google.com/u/3/ogw/ADGmqu_RtUB3lOe1TwHAnVoKiCUjsSmX3M0UU0cnFB5v=s32-c-mo',
                  label: 'Danielle Moreland',
                  email: 'danielle@morelands.net',
               },
               {
                  value: 'angelikamoreland',
                  photo: 'https://lh3.google.com/u/2/ogw/ADGmqu_27tkyuz1xbQptjxFy9poCkLAJ3WJKL8SSzOqC=s32-c-mo',
                  label: 'Angelika Moreland',
               },
            ],
         },
         upcoming: [],
         showAdvancedOptions: false,
         recurrence_expanded: false,
         optional_expanded: false,
      }
   },

   methods: {
      initPage() {
         this.formData = {
            notes: '',
            reminder: '',
            userId: '',
            sendTo: {
               inApp: true,
               email: false,
               mobile: false,
            },
            rrule: '',
            linked: {},
         }

         this.formHelpers = {
            time: '1000',
            starting: 'today',
            recurrence: 'once',
            recurrenceCustom: false,
            interval: 1,
            bySetPos: 0,
            wkst: { value: 'SU', label: 'Sunday' },
            byDay: [],
            byMonth: [],
            byMonthDay: '',
            byYearDay: '',
            byHour: '',
            byMinute: '',
            collection: 'medicines',
            linked: [],
            user: this.options.users[0],
         }
      },

      displayTime() {
         if (!this.formHelpers.time) return ''
         let hour = Number(this.formHelpers.time.substr(0, 2))
         let minute = Number(this.formHelpers.time.substr(2, 2))
         let second = Number(this.formHelpers.time.substr(4, 2))
         return moment()
            .hour(hour)
            .minute(minute)
            .second(second)
            .format('h:mm a')
      },

      displayDate() {
         if (!this.formHelpers.date) return ''
         let year = Number(this.formHelpers.date.substr(0, 4))
         let month = Number(this.formHelpers.date.substr(4, 2) - 1)
         let day = Number(this.formHelpers.date.substr(6, 2))
         return moment()
            .year(year)
            .month(month)
            .date(day)
            .format('ddd, MMM D, YYYY')
      },

      displayDateOptions(date) {
         return date >= moment().format('YYYY/MM/DD')
      },

      onClickStarting(option) {
         this.formHelpers.starting = option
      },

      onClickRecurrence(option) {
         this.formHelpers.recurrence = option
      },

      onDateInput(value) {
         this.formHelpers.starting = 'custom'
         this.formHelpers.date = value
      },

      onCancel() {
         this.closeScheduler()
         this.initPage()
      },

      onSubmit() {
         this.$refs.form.validate()
            .then(result => {
               if (result) {
                  this.handleSubmit(this.formData)
                  this.closeScheduler()
               }
            })
            .finally(() => {
               this.initPage()
            })
      },
   },

   watch: {
      'formHelpers.starting': function(value) {
         let hour = Number(this.formHelpers.time.substr(0, 2))
         let minute = Number(this.formHelpers.time.substr(2, 2))
         let second = Number(this.formHelpers.time.substr(4, 2))

         let startDate
         switch (value) {
            case 'today':
               startDate = moment()
                  .hour(hour)
                  .minute(minute)
                  .second(second)
                  .format('YYYYMMDD')
               this.formHelpers.date = startDate
               break
            case 'tomorrow':
               startDate = moment()
                  .add(1, 'd')
                  .hour(hour)
                  .minute(minute)
                  .second(second)
                  .format('YYYYMMDD')
               this.formHelpers.date = startDate
               break
            case 'custom':
               startDate = moment(this.formHelpers.date)
                  .hour(hour)
                  .minute(minute)
                  .second(second)
                  .format('YYYYMMDD')
               this.formHelpers.date = startDate
               break
         }
      },

      'formHelpers.date': function(value) {
         this.rrule.DTSTART.DATE = value
      },

      'formHelpers.time': function(value) {
         let hour = Number(value.substr(0, 2))
         let minute = Number(value.substr(2, 2))
         let second = Number(value.substr(4, 2))
         this.rrule.DTSTART.TIME = moment()
            .hour(hour)
            .minute(minute)
            .second(second)
            .format('HHmmss')
      },

      'formHelpers.recurrence': function(value) {
         switch (value) {
            case 'once':
               this.rrule.RRULE.COUNT = 1
               this.rrule.RRULE.FREQ = 'DAILY'
               this.recurrenceText = ''
               break

            case 'minutely':
               this.rrule.RRULE.COUNT = null
               this.rrule.RRULE.FREQ = 'MINUTELY'
               this.recurrenceText = 'minutes'
               break

            case 'hourly':
               this.rrule.RRULE.COUNT = null
               this.rrule.RRULE.FREQ = 'HOURLY'
               this.recurrenceText = 'hours'
               break

            case 'daily':
               this.rrule.RRULE.COUNT = null
               this.rrule.RRULE.FREQ = 'DAILY'
               this.recurrenceText = 'days'
               break

            case 'weekly':
               this.rrule.RRULE.COUNT = null
               this.rrule.RRULE.FREQ = 'WEEKLY'
               this.recurrenceText = 'weeks'
               break

            case 'monthly':
               this.rrule.RRULE.COUNT = null
               this.rrule.RRULE.FREQ = 'MONTHLY'
               this.recurrenceText = 'months'
               break

            case 'annually':
               this.rrule.RRULE.COUNT = null
               this.rrule.RRULE.FREQ = 'YEARLY'
               this.recurrenceText = 'years'
               break
         }
      },

      'formHelpers.interval': function(value) {
         this.rrule.RRULE.INTERVAL = value
      },

      'formHelpers.bySetPos': function(value) {
         if (value === '0' || value > 366 || value < -366) {
            this.rrule.RRULE.BYSETPOS = null
         } else {
            this.rrule.RRULE.BYSETPOS = value
         }
      },

      'formHelpers.wkst': function(value) {
         this.rrule.RRULE.WKST = value.value
      },

      'formHelpers.byDay': function(value) {
         let byDay = ''
         value.map(day => {
            if (byDay.length) byDay += ','
            byDay += day.value
         })
         this.rrule.RRULE.BYDAY = byDay
      },

      'formHelpers.byMonth': function(value) {
         let byMonth = ''
         value.map(month => {
            if (byMonth.length) byMonth += ','
            byMonth += month.value
         })
         this.rrule.RRULE.BYMONTH = byMonth
      },

      'formHelpers.byMonthDay': function(value) {
         this.rrule.RRULE.BYMONTHDAY = value
      },

      'formHelpers.byYearDay': function(value) {
         this.rrule.RRULE.BYYEARDAY = value
      },

      'formHelpers.byHour': function(value) {
         this.rrule.RRULE.BYHOUR = value
      },

      'formHelpers.byMinute': function(value) {
         this.rrule.RRULE.BYMINUTE = value
      },

      'formHelpers.user': function(value) {
         this.formData.userId = value.value
         if (this.formData.sendTo.mobile) this.formData.sendTo.mobile = !!value.mobile
         if (this.formData.sendTo.email) this.formData.sendTo.email = !!value.email
      },

      'formHelpers.collection': function() {
         this.formHelpers.linked = []
      },

      'formHelpers.linked': function(value) {
         this.formData.linked = { collection: this.formHelpers.collection, collectionId: value.value }
      },

      'rrule': {
         deep: true,
         handler() {
            let rruleString = 'DTSTART;TZID='
            rruleString += this.rrule.DTSTART.TZID
            rruleString += ':'
            rruleString += this.rrule.DTSTART.DATE
            rruleString += 'T'
            rruleString += this.rrule.DTSTART.TIME
            rruleString += 'Z'
            rruleString += '\n'
            rruleString += 'RRULE:FREQ='
            rruleString += this.rrule.RRULE.FREQ
            rruleString += ';INTERVAL='
            rruleString += this.rrule.RRULE.INTERVAL
            rruleString += ';WKST='
            rruleString += this.rrule.RRULE.WKST
            rruleString += ';BYSECOND='
            rruleString += this.rrule.RRULE.BYSECOND
            if (this.rrule.RRULE.BYSETPOS) {
               rruleString += ';BYSETPOS='
               rruleString += this.rrule.RRULE.BYSETPOS
            }
            if (this.rrule.RRULE.BYDAY) {
               rruleString += ';BYDAY='
               rruleString += this.rrule.RRULE.BYDAY
            }
            if (this.rrule.RRULE.BYMONTH) {
               rruleString += ';BYMONTH='
               rruleString += this.rrule.RRULE.BYMONTH
            }
            if (this.rrule.RRULE.BYMONTHDAY) {
               rruleString += ';BYMONTHDAY='
               rruleString += this.rrule.RRULE.BYMONTHDAY
            }
            if (this.rrule.RRULE.BYYEARDAY) {
               rruleString += ';BYYEARDAY='
               rruleString += this.rrule.RRULE.BYYEARDAY
            }
            if (this.rrule.RRULE.BYHOUR) {
               rruleString += ';BYHOUR='
               rruleString += this.rrule.RRULE.BYHOUR
            }
            if (this.rrule.RRULE.BYMINUTE) {
               rruleString += ';BYMINUTE='
               rruleString += this.rrule.RRULE.BYMINUTE
            }
            if (this.rrule.RRULE.COUNT) {
               rruleString += ';COUNT='
               rruleString += this.rrule.RRULE.COUNT
            }
            this.rruleString = rruleString
         },
      },

      'rruleString': function(value) {
         this.formData.rrule = value
         try {
            let rule = rrulestr(value)
            if (!rule) return false
            this.rruleText = rule.toText()

            let rruleString = 'DTSTART;TZID='
            rruleString += this.rrule.DTSTART.TZID
            rruleString += ':'
            rruleString += this.rrule.DTSTART.DATE
            rruleString += 'T'
            rruleString += this.rrule.DTSTART.TIME
            rruleString += 'Z'
            rruleString += '\n'
            rruleString += 'RRULE:FREQ='
            rruleString += this.rrule.RRULE.FREQ
            rruleString += ';INTERVAL='
            rruleString += this.rrule.RRULE.INTERVAL
            rruleString += ';WKST=SU'
            rruleString += ';BYSECOND='
            rruleString += this.rrule.RRULE.BYSECOND
            if (this.rrule.RRULE.BYSETPOS) {
               rruleString += ';BYSETPOS='
               rruleString += this.rrule.RRULE.BYSETPOS
            }
            if (this.rrule.RRULE.BYDAY) {
               rruleString += ';BYDAY='
               rruleString += this.rrule.RRULE.BYDAY
            }
            if (this.rrule.RRULE.BYMONTH) {
               rruleString += ';BYMONTH='
               rruleString += this.rrule.RRULE.BYMONTH
            }
            if (this.rrule.RRULE.BYMONTHDAY) {
               rruleString += ';BYMONTHDAY='
               rruleString += this.rrule.RRULE.BYMONTHDAY
            }
            if (this.rrule.RRULE.BYYEARDAY) {
               rruleString += ';BYYEARDAY='
               rruleString += this.rrule.RRULE.BYYEARDAY
            }
            if (this.rrule.RRULE.BYHOUR) {
               rruleString += ';BYHOUR='
               rruleString += this.rrule.RRULE.BYHOUR
            }
            if (this.rrule.RRULE.BYMINUTE) {
               rruleString += ';BYMINUTE='
               rruleString += this.rrule.RRULE.BYMINUTE
            }
            rruleString += ';COUNT='
            rruleString += this.rrule.RRULE.COUNT ? this.rrule.RRULE.COUNT : 5
            let sampleRule = rrulestr(rruleString)
            let sampling = sampleRule.all()
            let response = []
            sampling.map(sample => {
               let result = moment(sample).utc().format('ddd, MMM D, YYYY [at] h:mm a')
               response.push(result)
            })
            this.rruleSample = response
         } catch (err) {
            console.log('Unable to process RRULE', err)
         }
      },
   },

   beforeMount() {
      this.initPage()
   },
}
</script>

<style scoped>

</style>
