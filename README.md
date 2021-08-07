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

To deploy this docs website to production (in AWS), do the below

1. install hugo (google for instructions)

2. cd ~docs  (ensure you are in your docs repo)

3. sudo npm install -g postcss-cli

   sudo npm install autoprefixer
   
   sudo npm audit fix
    
4. Setup the aws credentials to use our aws account. This involves first downloading
   the aws-cli (or awscli) and executing "aws configure". This command will prompt for
   AWS Access Key ID, AWS Secret Key, Default Region name (use us-east-2) and
   Default output format (use json).  You will need to have the Access and Secret keys
   handy.

The above steps are onetime only

5. say "sudo hugo --minify" and "hugo deploy"
