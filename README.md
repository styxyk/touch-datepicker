# Datepicker

A datepicker Vue component. Compatible with Vue 2.x. Works nice on mobile devices.

- [Preview](#preview)
- [Install](#install)
- [Usage](#usage)
- [Props](#available-props)
- [Events](#events)
- [Date Formatting](#date-formatting)

## Preview

![alt text](https://github.com/styxyk/touch-datepicker/blob/master/preview.png "Preview")


## Install

Copy TouchDatePicker.vue component into your project

## Usage

``` javascript
import Datepicker from 'touch-datepicker';

export default {
  // ...
  components: {
    Datepicker
  }
  // ...
}
```

``` html
<datepicker></datepicker>
```

Using `v-model`
``` html
<datepicker v-model="state.date" name="uniquename"></datepicker>
```

Emits events
``` html
<datepicker v-on:change="saveModification"></datepicker>
```

Full example
``` html
<datepicker v-bind:language="en" v-bind:placeholder="Date of birth" v-bind:input-class="{'datepicker-input-reg': true, 'is-invalid-input': errors.has('event_date')}" v-bind:min="2000-01-01"  v-bind:max="2010-12-31" 
v-bind:data-vv-as="Please enter your birthdate" v-model="event_date" v-bind:v-validate="{required: true, date_format: 'YYYY-MM-DD'}" name="event_date" id="event_date" @change="saveChanges"></datepicker>
<span class="form-error" :class="{'is-visible': errors.has('event_date')}">{{ errors.first('event_date') }}</span>
```


## Available props

| Prop                          | Type            | Default     | Description                              |
|-------------------------------|-----------------|-------------|------------------------------------------|
| value                         | Date\|String    |             | Date value of the datepicker             |
| name                          | String          |             | Input name property                      |
| id                            | String          |             | Input id                                 |
| format-date                   | String          | DD MMMM YYYY| Date formatting string                   |
| language                      | String          | en          | Translation language used by moment      |
| v-validate                    | Object          |             | Validation rules used by vee-validate    |
| input-class                   | String\|Object  |             | CSS class applied to the input el        |
| placeholder                   | String          |             | Input placeholder text                   |
| data-vv-as                    | String          |             | See vee-validate: Errors localization    |
| max                           | String          |             | Max calendar date                        |
| min                           | String          |             | Min calendar date                        |


## Events

You can emit any js event as you would do on a DOM input


## Date formatting

#### String formatter

See https://momentjs.com/docs/#/parsing/string-format/

