Once this repo is cloned, one time do 'git submodule update --init --recursive' to initialize
the other infrastructure elements needed to create a docs UI

To see the docs UI, do

1. docker-compose build (you might have to install docker-compose)
2. docker-compose up
3. Open your web browser and type http://localhost:1313

The UI will automatically update itself when you update the docs. The docs are all in
the content/en directory and the pictures that the docs uses are all in the static/
directory. Some pictures have been generated from ppt files, those ppt files themseves
are in ppts/ directory.

To deploy this docs website to production, do the below

1. install hugo (google for instructions)
2. npm install -g postcss-cli
   npm install autoprefixer
   npm audit fix
3. Setup the aws credentials to use our aws account

The above steps are onetime only

4. say "hugo --minify" and "hugo deploy"
