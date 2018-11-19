# SmartCash Mempool Statistics

## Installation: Part 1

Create user with sudo rights called mempool.

    Install smarcashd by PPA
    sudo add-apt-repository ppa:smartcash/ppa -y
    sudo apt-get update
    sudo apt-get install smartcashd -y
    smartcash-cli stop

Edit the config file with nano and press control x to save and exit.
    nano smartcash.conf
      daemon=1
      rpcuser=mempool
      rpcpassword=mempool
    smartcashd

    sudo apt-get install git
    git clone https://github.com/smartcash/mempool

Edit `mempool.sh` to adapt paths as necessary, especially the path to 
smartcash-cli.  Add a smartcash.conf with rpcuser/rpcpassword settings to 
`/home/mempool/.smartcash`, to be able to use smartcash-cli.  You can test your
setup by running

    smartcash-cli getmempoolinfo

Install `mysql` and create a database. Then you can test your
configuration by running mempool.sh.  If you don't want to use mysql,
comment out the four lines starting with `open(SQL` at the end of
`mempool-sql.pl`.  In that case zooming and auto-update in the
webinterface won't work.

    sudo apt install mysql_server
    sudo mysql_secure_installation
    sudo mysql -u root -p <<EOF
    create database smart_mempool;
    grant all privileges on smart_mempool.* TO 'mempool'@'localhost' identified by 'mempool';
    grant select on smart_mempool.* TO 'www'@'localhost' identified by 'mempool';
    EOF
    cat > ~/.my.cnf <<EOF
    [client]
    user=mempool
    password=mempool
    EOF
    cd mempool
    perl mempool-create.pl | mysql smart_mempool
    ./mempool.sh

You are almost ready now.  Check that everything works.  There should be a
file `mempool.log` containing one line of statistics.  There should be
newly created files in `/dev/shm/mempool-smart` that contain the dynamic data the
webserver should serve.  If everything looks fine add the following crontab 
entry (using `crontab -e`):

    * * * * * /home/mempool/mempool/mempool.sh 

## Installation: Part 2 - Web service

Install a web server, copy files to web directory, and link data.

    sudo apt-get install apache2
    sudo nano /etc/apache2/sites_enabled/000-default.conf
      document path=/var/www/mempool
    (control x to save and quit)
    sudo mkdir /var/www/mempool
    sudo cp -r $HOME/mempool/web/* /var/www/html/mempool/
    ln -s /dev/shm/mempool-smart/*.js /var/www/html/mempool/queue/

Restart webserver for changes to take effect.
    sudo systemctl restart apache2.service

Start on startup or with crontab with @reboot
    service mysql start
    smartcashd
    
