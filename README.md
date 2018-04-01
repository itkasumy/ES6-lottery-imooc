1. mkdir 01-es6
2. cd 01-es6/
3. mkdir app
4. mkdir server
5. mkdir tasks
6. cd app/
7. mkdir css
8. mkdir js
9. mkdir views
10. mkdir js/class
11. touch js/class/test.js
12. touch js/index.js
13. touch views/error.ejs
14. touch views/index.ejs
15. cd ../server/
16. express -e .
17. npm install
18. cd ../tasks/
19. mkdir util
20. touch util/args.js
21. cd ..
22. npm init
23. touch .babelrc
24. touch gulpfile.babel.js
25. tasks/util/args.js

```js
import yargs from 'yargs'

const args = yargs
  .option('production', {
    boolean: true,
    default: false,
    describe: 'min all scripts'
  })
  .option('watch', {
    boolean: true,
    default: false,
    describe: 'watch all files'
  })
  .option('verbose', {
    boolean: true,
    default: false,
    describe: 'log'
  })
  .option('sourcemaps', {
    describe: 'force the creation of sourcemaps'
  })
  .option('port', {
    string: true,
    default: 8080,
    describe: 'server port'
  })
  .argv
```

26. touch tasks/scripts.js
27. tasks/scripts.js
  
```js
import gulp from 'gulp'
import gulpif from 'gulp-if'
import concat from 'gulp-concat'
import webpack from 'webpack'
import gulpWebpack from 'webpack-stream'
import named from 'vinyl-named'
import livereload from 'gulp-livereload'
import plumber from 'gulp-plumber'
import rename from 'gulp-rename'
import uglify from 'gulp-uglify'
import {log, colors} from 'gulp-util'
import args from './util/args'
```

28. cnpm install gulp gulp-if gulp-concat webpack webpack-stream vinyl-named gulp-livereload gulp-plumber gulp-rename gulp-uglify gulp-util yargs --save-dev
29. tasks/scripts.js

```js
gulp.task('scripts', () => {
  return gulp.src(['app/js/index.js'])
  .pipe(plumber({
    errorHandle: function () {
      
    }
  }))
  .pipe(named())
  .pipe(gulpWebpack({
    module: {
      loaders: [
        {
          test: /\.js$/,
          loader: 'babel'
        }
      ]
    }
  }), null, (err, stats) => {
    log(`Finished '${colors.cyan('scripts')}'`, stats.toString({
      chunks: false
    }))
  })
  .pipe(gulp.dest('server/public/js'))
  .pipe(rename({
    basename: 'cp',
    extname: '.min.js'
  }))
  .pipe(uglify({
    compress: {
      properties: false
    },
    output: {
      'quote_keys': true
    }
  }))
  .pipe(gulp.dest('server/public/js'))
  .pipe(gulpif(args.watch, livereload()))
})
```

30. touch tasks/pages.js
31. tasks/page.js

```js
import gulp from 'gulp'
import gulpif from 'gulp-if'
import livereload from 'gulp-livereload'
import args from './util/args'

gulp.task('pages', () => {
  return gulp.src('app/**/*.ejs')
    .pipe(gulp.dest('server'))
    .pipe(gulpif(args.watch, livereload()))
})
```

32. touch tasks/css.js
33. tasks/css.js

```js
import gulp from 'gulp'
import gulpif from 'gulp-if'
import livereload from 'gulp-livereload'
import args from './util/args'

gulp.task('css', () => {
  return gulp.src('app/**/*.css')
    .pipe(gulp.dest('server/public'))
})
```

34. touch tasks/server.js
35. tasks/server.js

```js
import gulp from 'gulp'
import gulpif from 'gulp-if'
import liveserver from 'gulp-live-server'
import args from './util/args'

gulp.task('serve', (cb) => {
  if (!args.watch) return cb()
  var server = liveserver.new(['--harmony', 'server/bin/www'])
  server.start()

  gulp.watch(['server/public/**/*.js', 'server/views/**/*.ejs'], function (file) {
    server.notify.apply(server, [file])
  })

  gulp.watch(['server/routes/**/*.js', 'server/app.js'], function () {
    server.start.bind(server)()
  })
})
```

36. touch tasks/browser.js
37. tasks/browser.js

```js
import gulp from 'gulp'
import gulpif from 'gulp-if'
import gutil from 'gulp-util'
import args from './util/args'

gulp.task('browser', (cb) => {
  if (!args.watch) return cb()
  gulp.watch('app/**/*.js', ['scripts'])
  gulp.watch('app/**/*.ejs', ['pages'])
  gulp.watch('app/**/*.css', ['css'])
})
```