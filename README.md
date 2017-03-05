# gulp-futurejs-compile

[![npm](https://nodei.co/npm/gulp-futurejs-compile.svg?downloads=true)](https://nodei.co/npm/gulp-futurejs-compile/)

> Compile FutureScript as part of your Gulp build process.

## Usage

```js
var fus = require('gulp-futurejs-compile');

gulp.task('futurejs', function() {
  gulp.src('./src/*.fus')
    .pipe(fus(opts).on('error', gutil.log))
    .pipe(gulp.dest('./public/'))
});
```

### Error handling

`gulp-futurejs-compile` will emit an error for cases such as invalid Marko syntax. If uncaught, the error will crash gulp.

You will need to attach a listener (i.e. `.on('error')`) for the error event emitted by `gulp-futurejs-compile`:

```javascript
var fusStream = fus();

// Attach listener
fusStream.on('error', function(err) {});
```

In addition, you may utilize [gulp-util](https://github.com/wearefractal/gulp-util)'s logging function:

```javascript
var gutil = require('gulp-util');

// ...

var fusStream = fus();

// Attach listener
fusStream.on('error', gutil.log);

```

Since `.on(...)` returns `this`, you can make you can compact it as inline code:

```javascript

gulp.src('./src/*.fus')
  .pipe(fus({preserveWhitespace: true}).on('error', gutil.log))
  // ...
```

## Options

The options object supports the same [options](http://fusjs.com/docs/fus/javascript-api/#require'fuscompiler') as the standard FutureScript compiler

Input object:
```
{
    code: <string>,
    path: <string>,
    level: <OutputLevel>,
    generatesStyles: <boolean>,
    outputCodeReadability: <number>
}
```

Either `code` or `path` (or both) must be set.

`path` means the path of the file the code belongs to (or you want the code to behave as if

it is located at the path, for source maps and other use).

If `code` is set, it won't read code from `path`, otherwise it will read code from `path`.

`level` is optional defaulting to `OutputLevel.compile`.

`generatesStyles` is optional defaulting to false. If true, then `level` can't be less than

`OutputLevel.exports`. It categorizes code parts into styles. Useful in syntax highlighting.

`outputCodeReadability` is optional defaulting to 0. It must be between 0 (including) and 1 (including),
for example, `0.3`, `0.8`.

Output object:

```
{
    version: <string>,
    exports: <string array>,
    styles: <array of arrays>,
    targets: <Target array>
}
```

`targets` is an array of objects of virtual type `Target`.
`Target` type structure:

```
{
    code: <string>,
    codeType: <OutputCodeType>,
    sourceMap: <string>
}
```

`sourceMap` will be absent if `level` < `OutputLevel.sourceMap`.
The whole `targets` field will be absent if `level` < `OutputLevel.compile`.

`exports` will be absent if `level` < `OutputLevel.exports`. This property lists all exports' names
(note that the name of the default export is "default"). Will be an empty array if no export.

`styles` will be absent if `generatesStyles` is false. If present, it's
an array of arrays.
A `styles` example is [[0, 1], [3, 0]], which means the first segment's start index is 0
(this should always be 0) and its style is 1 (which means keyword), and the second
segment's start index is 3 and its style is 0 (which means normal). Style is a
number instead of a real color name, because the real color should be customizable,
also because sometimes we use underline, bold, italic, etc. to show a style.
We even don't use names `keyword` or `normal` because it's sometimes hard to define
a kind in accurate, irrevocable words. I can now only confirm the use of the first 4 styles:
0: normal, 1: keyword, 2: string, 3: comment
It can't be larger than 15, except for extended style. Is the limit too small? No, it's
by design, because if there're too many then the meanings will be less likely to be fixed.

Note: the `targets` array always has only 1 element. Why it's an array is for future use.

Note: Besides main style, there's extended style: a combination of different categories.

This is useful when adding `italic` or `underline` to a code fragment that already has many styles.

It uses flags, each of which has 16 possible values, so it's compatible with the main style.

The first category (bit 0-3) is the main style (note that it can also be displayed as italic
or underline, depending on the theme).

The second category (bit 4-7) is mainly for italic (only two lowest values used).

The third category (bit 8-11) is mainly for underline (only two lowest values used).

There may be additional categories for future use.

For example, if the style is `16`, it means `normal, italic`
If the style is `274`, it means `string, italic, underline`

## TODO

Fully comply with [Gulp plugin guidelines](https://github.com/gulpjs/gulp/blob/master/docs/writing-a-plugin/guidelines.md) AKA write some tests

## License

MIT License
