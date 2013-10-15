ansible-lssd
============

Large scale server deploys using BitTorrent and the BitTornado library by [Murder](https://github.com/lg/murder).

DESCRIPTION
-----------
ansible-lssd  is a method of using Bittorrent (Powered by [Murder](https://github.com/lg/murder)) to distribute files to a large amount
of servers within a production environment. 


QUICK START
-----------

For the impatient, `sudo pip install ansible` :
  ```bash:
  # install pip (Python package manager) and ansible
  $ sudo easy_install pip
  $ sudo pip install ansible
  ```

and add these lines to your group_vars/all:
  ```YAML:group_vars/all
  # deploy tag
  tag: Deploy1
  
  # path
  seeder_files_path:  ~/builds
  destination_path:   /directorypath/hoge/
  ```


HOW IT WORKS
------------

Same as the "[HOW IT WORKS](https://github.com/lg/murder/blob/master/README.md#how-it-works)" of Murder.


CONFIGURATION AND USAGE
-----------------------

You define `tracker`, `seeder` and `peer` server to inventory (./production) file.

All involved servers must have python and pizg installed and the related murder
support files (BitTornado, etc.). To upload the support files to the
tracker, seeder, and peers, run:

  ```bash:
  $ ansible-playbook -i prodction setup.yml
  ```

By default, these will go in `/usr/local/murder` in your apps deploy directory. 
Override this by setting the variable `remote_murder_path`. 

Before deploying, you must start the tracker:

  ```bash:
  $ ansible-playbook -i prodction start_tracker.yml
  ```

At this point you should be able to deploy normally:

  ```bash:
  $ ansible-playbook -i prodction deploy.yml
  ```


MANUAL USAGE (ansible-lssd without a deploy strategy)
-----------------------------------------------------

Modify a ./production and ./group_vars/all, manually define servers:

./production:
  ```INI:production
  # ansible host
  [ansible_host]
  localhost ansible_connection=local
  
  # tracker node
  [tracker]
  10.0.0.1
  
  # seeder node
  [seeder]
  10.0.0.1
  
  # peer nodes
  [peer]
  10.1.1.1
  10.1.1.2
  10.1.1.3
  ```

group_vars/all:
  ```YAML:group_vars/all
  # deploy tag
  tag: Deploy1
  
  # path
  seeder_files_path:  ~/builds
  destination_path:   /opt/hoge  # or some other directory
  ```

To distribute a directory of files, first make sure that murder is set
up on all hosts, then manually run the murder cap tasks:

1. Start the tracker:

  ```bash:
  $ ansible-playbook -i prodction start_tracker.yml
  ```

2. Create a torrent from a directory of files on the seeder, and start seeding:

  ```bash:
  # copy to seeder node
  $ scp -r ./builds 10.0.0.1:~/builds
  
  # create torrent file
  $ ansible-playbook -i prodction create_torrent.yml
  
  # start seeding
  $ ansible-playbook -i prodction start_seeder.yml
  ```

3. Distribute the torrent to all peers:

  ```bash:
  $ ansible-playbook -i prodction deploy.yml
  ```

4. Stop the seeder and tracker:

  ```bash:
  $ ansible-playbook -i prodction stop_seeder_and_tracker.yml
  ```

When this finishes, all peers will have the files in /opt/hoge/Deploy1


main yamls REFERENCE
--------------------

* `create_torrent.yml:`
  * Create torrent file on seeder node.
* `deploy.yml:`
  * Deploy files on peer nodes.
* `rm_tgz.yml:`
  * Delete the file deployment of targz in all peer node.
* `setup.yml:`
  * Install the software required for each node.
* `site.yml:`
  * Run all Playbook deploy from the setup.
* `start_seeder.yml:`
  * start seeding.
* `start_tracker.yml:`
  * start tracker.
* `stop_all_peers.yml:`
  * stop all peer client.
* `stop_seeder_and_tracker.yml:`
  * stop seeding and tracker.





