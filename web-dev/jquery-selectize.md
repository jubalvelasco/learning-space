# Selectize Notes

Example subject is a dropdown of languages

## Quick Note - Cloning Objects

```js
const userDetails = {
  name: "John Doe",
  age: 14,
  verified: false
};

let cloneUser = { ...userDetails }; // shallow copy: nested objects/arrays are still shared

// deep copy (modern browsers / Node.js 17+)
let deepCloneUser = structuredClone(userDetails);
```

## remove option

Should match a `value=?`

```js
jQuery('#_emp_language1')[0].selectize.removeOption('Arabic')
```

## get current selected value

will return an empty string (`''`) if there is no selection (for a multi-select it returns an array instead, empty if nothing is selected).

```js
jQuery('#_emp_language1')[0].selectize.getValue()
```

## get all options

```js
jQuery('#_emp_language1')[0].selectize.options
```

## get specific options

```js
jQuery('#_emp_language1')[0].selectize.options['Bulgarian']
```

## update an option - can disable here

```js
jQuery('#_emp_language1')[0].selectize.updateOption('Arabic', {
    text: 'Arabic', value: 'Arabic', disabled: true, $order: 1
});
```
