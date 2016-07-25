aspects_topbeat
=========

Install and configure Elastic.co's topbeat process and system resources watcher.

Requirements
------------

Set ```hash_behaviour=merge``` in your ansible.cfg file.

Role Variables
--------------

See docs/etc-topbeat-topbeat.default.yml for the default topbeat.yml file that is installed by the apt repository. Reference it when you are unsure of syntax.

Remember that indenting is very important. So, make sure your blocks of text in the various vars that use them, are indented correctly when the topbeat.yml file is created on your server.

* aspects_topbeat_enabled
  * True or False. If True, actually use this role. If False, skip everything in this role.
  * Default: False
* aspects_topbeat_install
  * True or False. If True, run installation tasks. If False, skip running installation tasks. The install<Debian|Redhat>.yml file.
  * Default: False
* aspects_topbeat_repository
  * A dictionary telling the role what information to use when setting up the Beats package repository.
* aspects_topbeat_repository.Debian
  * A subdictionary consisting of ```value```, ```path```, and ```key_url```. ```value``` contains the deb line to add to apt config. ```path``` contains the path to the file you want to add the deb line to. ```key_url``` is  the url of the repositories GPG key.
  * Defaults
    * ```value```: "deb https://packages.elastic.co/beats/apt stable main"
    * ```path```: "/etc/apt/sources.list.d/beats.list"
    * ```key_url```: "https://packages.elasticsearch.org/GPG-KEY-elasticsearch"
* aspects_topbeat_repository.RedHat
  * A subdictionary consisting of ```value```, ```path```, and ```key_url```. ```value``` contains a multiline string with the yum repository information. ```path``` contains the path to the file you want to add the ```value``` to. ```key_url``` is  the url of the repositories GPG key.
  * Defaults
    * value: |
      [beats]
      name=Elastic Beats Repository
      baseurl=https://packages.elastic.co/beats/yum/el/$basearch
      enabled=1
      gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
      gpgcheck=1
    * path: "/etc/yum.repos.d/beats.repo"
    * key_url: "https://packages.elastic.co/GPG-KEY-elasticsearch"
* aspects_topbeat_output
  * A dictionary of output configuration. Each item is a multiline string. If you want to disable a output that is your group config, just replace it with an empty string in your node config. This only works if you set ```hash_behaviour=merge``` in your ansible.cfg file.
  * Currently only ```aspects_topbeat_output.logstash``` and ```aspects_topbeat_output.elasticsearch``` are supported. If you need another type of output, you will need to modify the topbeat.yml.j2 template.
  * Default: {}
* aspects_topbeat_shipper
  * A dictionary configuring shipper behavior. See the example file for how you should use this. ```name```, ```tags```, ```ignore_outgoing```, ```refresh_topology_freq```, ```topology_expire```, ```geoip``` are all dictionary items. 
  * All items details can be found in docs/etc-topbeat-topbeat.default.yml. 
  * Note that ```geoip``` is a multiline string. Just put the config in as a block of text.
* aspects_topbeat_logging
  * A dictionary containing values for setting up Beats logging configuration. ```to_syslog```, ```to_files```, ```files.use_files```, ```files.path```, ```files.rotateeverybytes```, ```files.keepfiles```, ```selectors```, and ```level``` are the valid variables.
  * Defaults: this configuration is optional. The defaults topbeat uses are generally what you want. The only time you should need to configure is if you do not want to sent logs to syslog, or are using topbeat on Windows.
  * Default: undefined

Dependencies
------------

None at this time.

Example Playbook
----------------

    - hosts: webservers
      vars: 
        aspects_topbeat_install: False
        aspects_topbeat_enabled: True
        aspects_topbeat_prospectors:
          syslog: |
            - paths:
                - /var/log/syslog
                - /var/log/messages
              encoding: plain
              input_type: log
              fields:
                customfieldtest: "This is a test."
                testtwo: "Another field."
              fields_under_root: false
          apache: |
            - paths:
                - /var/log/apache2/*.log
              encoding: plain
              input_type: log
              fields:
                customfieldtest: "This is a apache2."
                testtwo: "all apache logs."
              fields_under_root: false
        aspects_topbeat_output:
          logstash: |
            hosts: ["logstashindexer01:5043"]
        aspects_topbeat_shipper:
          name: "{{ ansible_fqdn }}"
          tags: '["newtag"]'
          ignore_outgoing: true
          refresh_topology_freq: 10
          topology_expire: 15
          geoip: |
            paths:
              - "/usr/share/GeoIP/GeoLiteCity.dat"
              - "/usr/local/var/GeoIP/GeoLiteCity.dat"
        aspects_topbeat_logging:
          to_syslog: True
          to_files: False
          files:
            use_files: True
            path: /var/log/mybeat
            rotateeverybytes: 10485760
            keepfiles: 7
          selectors: '[ "*"]'
          level: error
      roles:
         - aspects_topbeat

License
-------

MIT
