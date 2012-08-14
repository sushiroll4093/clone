So, you have the app running in production...what about updating it to the latest version?

Here are my notes (haven't been fully tested - in the process):

These notes assume you followed the Production Start and we will use the same folder structure.
cd ~/canvas
sudo git pull main-origin stable
cp /var/rails/canvas/config /var/rails/canvas/config-old
sudo cp -av * /var/rails/canvas
sudo bundle install
sudo /etc/init.d/apache2 restart
Try It
tail -n 1000 /var/rails/canvas/log/production.log 
Errors?
Try
sudo $GEM_HOME/bin/bundle exec rake canvas:compile_assets
sudo bundle exec rake db:migrate
sudo touch Gemfile.lock
sudo chown -R canvasuser config/environment.rb log tmp public/assets \
                                                               public/stylesheets/compiled Gemfile.lock