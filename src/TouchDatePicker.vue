<template>
    <div class="date-container">
        <input type="text" readonly @focus="showCalendar" v-model="outputDate" :placeholder="placeholder" v-bind="$attrs" :id="id+'-output'" :class="inputClass" />
        <input type="hidden" @change="inputChangeDesktop" v-validate="vValidate" v-model="modelValue" :name="name" :id="id" :data-vv-as="dataVvAs" />

        <div class="date-select" v-show="calendarOpen">
            <div class="popup" onwheel="event.preventDefault()">
                <span class="tip"></span>
                <div class="select day" @wheel="dayWheel" @touchstart="fingerStart" @touchmove="fingerMoved('day', $event)">
                    <a @click="dayUp" class="btn-arrow btn-up"><i class="icon icon-up"></i></a>
                    <div>
                        <span class="num">{{ formDay }}</span>
                        <span class="text">{{ formDayName }}</span>
                        </div>
                    <a @click="dayDown" class="btn-arrow btn-down"><i class="icon icon-down"></i></a>
                </div>
                <div class="select month" @wheel="monthWheel" @touchstart="fingerStart" @touchmove="fingerMoved('month', $event)">
                    <a @click="monthUp" class="btn-arrow btn-up"><i class="icon icon-up"></i></a>
                    <div>
                        <span class="num">{{ formMonth }}</span>
                        <span class="text">{{ formMonthName }}</span>
                        </div>
                    <a @click="monthDown" class="btn-arrow btn-down"><i class="icon icon-down"></i></a>
                </div>
                <div class="select year" @wheel="yearWheel" @touchstart="fingerStart" @touchmove="fingerMoved('year', $event)">
                    <a @click="yearUp" class="btn-arrow btn-up"><i class="icon icon-up"></i></a>
                    <div>
                        <span class="num">{{ formYear }}</span>
                        </div>
                    <a @click="yearDown" class="btn-arrow btn-down"><i class="icon icon-down"></i></a>
                </div>
                <div class="buttons">
                    <a @click="hideCalendar" class="btn-cancel"><span class="icon">&times;</span></a>
                    <a @click="clearDateButton" class="btn-del"><span class="icon">&olarr;</span></a>
                    <a @click="acceptButton" class="btn-ok"><span class="icon">&check;</span></a>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
/* eslint-disable */
    import moment from 'moment'

    export default {
        props: {
            value: [Date,String], //value of hidden input in format YYYY-MM-DD
            name: String, //name of validated element
            id: {
                type: String,
                required: true
            },  //id of validated element
            formatDate: {
                type: String,
                default: "DD MMMM YYYY"
            }, //output format date
            language: String, //language: en, de, etc
            vValidate: Object, //v-validate rules
            inputClass: [String, Object], //class applied to the output element
            placeholder: String, //displayed element placeholder
            dataVvAs: String, //data-vv-as from vee-validate
            max: String, //max calendar date
            min: String //min calendar date
        },
        //inject: ['$validator'],
        data() {
            return {
                calendarOpen: false, //if popup calendar is visible
                maxDate: '', //max popup calendar date
                minDate: '', //min popup calendar date
                outputElement: false, //visible input element - which displays formatted date
                hiddenElement: false, //hidden input element - which contains date in format YYYY-MM-DD
                selectDate: false, //date used in popup calendar
                calendarLanguage: this.language ? this.language : 'en', //errors and dates language
                modelValue: this.value, //value of hidden input, is needed for validation to work
                lastClientY: false, //touchmove event,
            }
        },
        inheritAttrs: false,
        watch: {
            value(newVal) {
                this.modelValue = newVal;
            }
        },
        created() {
            moment.locale(this.calendarLanguage);

            if (this.max) {
                this.maxDate = moment(this.max);
            }

            if (this.min) {
                this.minDate = moment(this.min);
            }

            this.resetSelectDate();
        },
        mounted() {
            this.outputElement = this.$el.querySelector(`#${this.id}-output`);

            this.hiddenElement = this.$el.querySelector(`#${this.id}`);

            let newTop = this.outputElement.offsetHeight+12;

            this.$el.querySelector('.date-select').style.top = newTop + 'px';
        },
        computed: {
            outputDate() {
                if (this.value) {
                    return moment(this.value).format(this.formatDate);
                }

                return ''
            },
            formDay() {
                return this.selectDate.format('DD')
            },
            formMonth() {
                return this.selectDate.format('MM')
            },
            formYear() {
                return this.selectDate.format('YYYY')
            },
            formDayName() {
                return this.selectDate.format('dddd')
            },
            formMonthName() {
                return this.selectDate.format('MMMM')
            }
        },
        methods: {
            fingerStart(e) {
                this.lastClientY = e.touches[0].clientY;
            },
            fingerMoved(type, e) {
                if (e.touches[0].clientY > this.lastClientY) {
                    if (type == 'month') {
                        this.monthDown()
                    }
                    else if (type == 'day') {
                        this.dayDown()
                    }
                    else if (type == 'year') {
                        this.yearDown()
                    }
                } else if (e.touches[0].clientY < this.lastClientY) {
                    if (type == 'month') {
                        this.monthUp()
                    }
                    else if (type == 'day') {
                        this.dayUp()
                    }
                    else if (type == 'year') {
                        this.yearUp()
                    }
                }

                this.lastClientY = e.touches[0].clientY;
            },
            addOutsideClickListener() {
                document.addEventListener('click', this.clickOutside, false)
            },
            removeOutsideClickListener() {
                document.removeEventListener('click', this.clickOutside, false)
            },
            clickOutside(e) {
                if (this.$el && !this.$el.contains(e.target)) {
                    this.hideCalendar(e)
                }
            },

            inputChangeMobile(e) {
                this.$emit('change', e);

                this.$emit('input', this.$el.value);
            },
            inputChangeDesktop(e) {
                setTimeout(() => {
                    this.$emit('change', e);
                }, 0)
            },

            resetSelectDate() {
                if (this.value) {
                    this.assignSelectDate(moment(this.value));
                } else {
                    this.assignSelectDate(moment());
                }
            },
            assignSelectDate(newDate) {
                if (typeof this.maxDate == 'object' && this.maxDate.isBefore(newDate)) {
                    newDate = this.maxDate;
                }

                if (typeof this.minDate == 'object' && this.minDate.isAfter(newDate)) {
                    newDate = this.minDate;
                }

                this.selectDate = newDate;
            },

            showCalendar() {
                if (this.calendarOpen) {
                    return;
                }

                this.addOutsideClickListener();

                this.resetSelectDate();

                this.calendarOpen = true;
            },
            hideCalendar(e) {
                e.preventDefault();

                this.removeOutsideClickListener();

                this.calendarOpen = false;
            },

            acceptButton(e) {
                this.$emit('input', this.selectDate.format('YYYY-MM-DD'));

                this.hiddenElement.dispatchEvent(new Event('change'))

                this.hideCalendar(e);
            },
            clearDateButton(e) {
                this.$emit('input', '');

                this.hiddenElement.dispatchEvent(new Event('change'))

                this.hideCalendar(e);
            },

            dayUp() {
                this.assignSelectDate(moment(this.selectDate).add(1, 'days'))

                this.outputElement.focus()
            },
            dayDown() {
                this.assignSelectDate(moment(this.selectDate).subtract(1, 'days'))

                this.outputElement.focus()
            },
            monthUp() {
                this.assignSelectDate(moment(this.selectDate).add(1, 'months'))

                this.outputElement.focus()
            },
            monthDown() {
                this.assignSelectDate(moment(this.selectDate).subtract(1, 'months'))

                this.outputElement.focus()
            },
            yearUp() {
                this.assignSelectDate(moment(this.selectDate).add(1, 'years'))

                this.outputElement.focus()
            },
            yearDown() {
                this.assignSelectDate(moment(this.selectDate).subtract(1, 'years'))

                this.outputElement.focus()
            },
            dayWheel(e) {
                if (e.deltaY < 0) {
                    this.dayUp()
                }
                else if (e.deltaY > 0) {
                    this.dayDown()
                }
            },
            monthWheel(e) {
                if (e.deltaY < 0) {
                    this.monthUp()
                }
                else if (e.deltaY > 0) {
                    this.monthDown()
                }
            },
            yearWheel(e) {
                if (e.deltaY < 0) {
                    this.yearUp()
                }
                else if (e.deltaY > 0) {
                    this.yearDown()
                }
            }
        }
    }
</script>

<style>
    .date-container {
        position: relative;
        display: inline-block;
    }
    .date-select {
        position: absolute;
        font-family: sans-serif;
        font-weight: 400;
    }
    .date-select .popup {
        position: absolute;
        left: 0;
        top: 0;
        width: 302px;
        height: 151px;
        padding: 1px;
        z-index: 15;
        background: #FFF;
        border: 1px solid #CCC;
        -webkit-box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.3);
        box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.3);
        -webkit-box-sizing: content-box;
        -moz-box-sizing: content-box;
        box-sizing: content-box;
    }
    .date-select .tip {
        position: absolute;
        left: 12px;
        top: -12px;
        z-index: 5;
        width: 0;
        height: 0;
        border-left: 10px solid transparent;
        border-right: 10px solid transparent;
        border-bottom: 12px solid #CCC;
    }
    .date-select .tip:before {
        content: ' ';
        position: absolute;
        left: -8px;
        top: 2px;
        z-index: 20;
        width: 0;
        height: 0;
        border-left: 8px solid transparent;
        border-right: 8px solid transparent;
        border-bottom: 10px solid #FFF;
    }
    .date-select .select {
        width: 100px;
        height: 100px;
        float: left;
        margin-right: 1px;
        color: #222;
        position: relative;
        cursor: default;
        touch-action: none;
    }
    .date-select .select.year {
        margin-right: 0;
    }
    .date-select .select > div {
        position: absolute;
        left: 0;
        top: 20px;
        text-align: center;
        width: 100%;
    }
    .date-select .select span {
        display: block;
        margin: 0 10px 0;
    }
    .date-select .select span.num {
        font-size: 34px;
        color: #555;
        line-height: 45px;
    }
    .date-select .select span.text {
        font-size: 12px;
        text-transform: uppercase;
        line-height: 15px;
    }
    .date-select .select:hover {
        background: #1196CE;
        color: #FFF;
    }
    .date-select .select:hover span {
        color: #FFF;
    }
    .date-select .select a.btn-arrow {
        position: absolute;
    }
    .date-select .select a.btn-arrow.btn-up {
        left: 0;
        top: 0;
        width: 100%;
        height: 20px;
    }
    .date-select .select a.btn-arrow.btn-down {
        left: 0;
        bottom: 0;
        width: 100%;
        height: 20px;
    }
    .btn-arrow, .btn-cancel, .btn-del, .btn-ok {
        cursor: pointer;
    }
    .icon {
        font-size: 24px;
    }
    .date-select .select .icon {
        display: block;
        margin: 7px auto;
        width: 0;
        height: 0;
        -webkit-transition: opacity 0.3s;
        -moz-transition: opacity 0.3s;
        -ms-transition: opacity 0.3s;
        -o-transition: opacity 0.3s;
        transition: opacity 0.3s;
    }
    .date-select .select .icon.icon-up {
        border-left: 5px solid transparent;
        border-right: 5px solid transparent;
        border-bottom: 6px solid #ccc;
    }
    .date-select .select .icon.icon-down {
        border-left: 5px solid transparent;
        border-right: 5px solid transparent;
        border-top: 6px solid #ccc;
    }
    .date-select .select:hover .icon {
        filter: alpha(opacity=100);
        opacity: 1;
    }
    .date-select .select:hover .icon.icon-up {
        border-bottom: 6px solid #87CAE6;
    }
    .date-select .select:hover .icon.icon-down {
        border-top: 6px solid #87CAE6;
    }
    .date-select .select a.btn-arrow:hover .icon.icon-up {
        border-bottom-color: #FFF;
    }
    .date-select .select a.btn-arrow:hover .icon.icon-down {
        border-top-color: #FFF;
    }
    .date-select .buttons {
        margin: 111px auto 0;
        width: 302px;
    }
    .date-select .buttons a {
        display: block;
        width: 100px;
        height: 40px;
        color: #343434;
        text-align: center;
        line-height: 40px;
        font-size: 18px;
        float: left;
    }
    .date-select .buttons a:hover {
        color: #FFF;
    }
    .date-select .buttons a.btn-ok {
        margin-left: 1px;
    }
    .date-select .buttons a.btn-del {
        margin-left: 1px;
    }
    .date-select .buttons a.btn-del:hover {
        background: #D95005;
    }
    .date-select .buttons a.btn-ok:hover {
        background: #69B22C;
    }
    .date-select .buttons a.btn-cancel:hover {
        background: #D95005;
    }
</style>
