apache
======

Ansible role which helps to install and configure Apache web server.

The configuraton of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely universal.
See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
# Example of how to use it with the default settings
- name: Example 1
  hosts: host1
  roles:
    - apache

# Example of how to change server binding and admin e-mail
- name: Example 2
  hosts: host2
  vars:
    # Bind the server to the address 192.168.1.10 and the port 8080
    apache_config_core_Listen: 192.168.1.10:8080
    # Admin e-mail
    apache_config_base_ServerAdmin: admin@example.com
  roles:
    - apache

# Example of how to load another Apache modules
- name: Example 3
  hosts: host3
  vars:
    apache_config_modules__custom:
      # That's the first module
      - LoadModule:
        - my_first_module
        - modules/mod_my_first.so
      # That's the second module
      - LoadModule:
        - my_second_module
        - modules/mod_my_second.so
  roles:
    - apache

# Example of how to add custom settings into the httpd.conf file
- name: Example 4
  hosts: host4
  vars:
    # Setting added at the end of the file
    apache_config__custom:
      - options:
        # Listen on HTTPS port
        - Listen: 443
  roles:
    - apache

# Example of how to create a new vhost file
- name: Example 5
  hosts: host5
  vars:
    # Create new vhost for www.example.com
    apache_vhost_config__custom:
      # This is the name of the vhost file
      www.example.com:
        content:
          - sections:
            - name: VirtualHost
              param: "*:80"
              content:
                - options:
                  - DocumentRoot: /www/example1
                  - ServerName: www.example.com
                - options:
                  # This is actually a comment
                  - "#": Other directives here ...
  roles:
    - apache
```

This role requires [Config
Encoders](https://github.com/jtyr/ansible/blob/jtyr-config_encoders/lib/ansible/plugins/filter/config_encoders.py)
which must be configured in the `ansible.cfg` file like this:

```
[defaults]

filter_plugins = ./plugins/filter/
```

Where the `./plugins/filter/` containes the `config_encoders.py` file.


Role variables
--------------

```
# Name of the Apache service
apache_service: httpd

# Apache user and group
apache_user: apache
apache_group: apache

# Package to be installed (exact version can be specified here)
apache_pkg: httpd

# List of additional packages to be installed (e.g. mod_ssl)
apache_additional_pkgs: []

# Path to the httpd.conf file
apache_httpd_path: /etc/httpd/conf/httpd.conf


# Core
apache_config_core_ServerTokens: OS
apache_config_core_ServerRoot: /etc/httpd
apache_config_core_PidFile: run/httpd.pid
apache_config_core_Timeout: 60
apache_config_core_KeepAlive: "Off"
apache_config_core_MaxKeepAliveRequests: 100
apache_config_core_KeepAliveTimeout: 15
apache_config_core_Listen: 80

apache_config_core__default:
  - ServerTokens: "{{ apache_config_core_ServerTokens }}"
  - ServerRoot: "{{ apache_config_core_ServerRoot }}"
  - PidFile: "{{ apache_config_core_PidFile }}"
  - Timeout: "{{ apache_config_core_Timeout }}"
  - KeepAlive: "{{ apache_config_core_KeepAlive }}"
  - MaxKeepAliveRequests: "{{ apache_config_core_MaxKeepAliveRequests }}"
  - KeepAliveTimeout: "{{ apache_config_core_KeepAliveTimeout }}"
  - Listen: "{{ apache_config_core_Listen }}"

apache_config_core__custom: []

apache_config_core:
  - options: "{{
      apache_config_core__default +
      apache_config_core__custom
    }}"


# MPM
apache_config_mpm_prefork_StartServers: 8
apache_config_mpm_prefork_MinSpareServers: 5
apache_config_mpm_prefork_MaxSpareServers: 20
apache_config_mpm_prefork_ServerLimit: 256
apache_config_mpm_prefork_MaxClients: 256
apache_config_mpm_prefork_MaxRequestsPerChild: 4000

apache_config_mpm_prefork__default:
  - StartServers: "{{ apache_config_mpm_prefork_StartServers }}"
  - MinSpareServers: "{{ apache_config_mpm_prefork_MinSpareServers }}"
  - MaxSpareServers: "{{ apache_config_mpm_prefork_MaxSpareServers }}"
  - ServerLimit: "{{ apache_config_mpm_prefork_ServerLimit }}"
  - MaxClients: "{{ apache_config_mpm_prefork_MaxClients }}"
  - MaxRequestsPerChild: "{{ apache_config_mpm_prefork_MaxRequestsPerChild }}"

apache_config_mpm_prefork__custom: []

apache_config_mpm_prefork:
  - name: IfModule
    param: prefork.c
    content:
      - options: "{{
          apache_config_mpm_prefork__default +
          apache_config_mpm_prefork__custom
        }}"

apache_config_mpm_worker_StartServers: 4
apache_config_mpm_worker_MaxClients: 300
apache_config_mpm_worker_MinSpareThreads: 25
apache_config_mpm_worker_MaxSpareThreads: 75
apache_config_mpm_worker_ThreadsPerChild: 25
apache_config_mpm_worker_MaxRequestsPerChild: 0

apache_config_mpm_worker__default:
  - StartServers: "{{ apache_config_mpm_worker_StartServers }}"
  - MaxClients: "{{ apache_config_mpm_worker_MaxClients }}"
  - MinSpareThreads: "{{ apache_config_mpm_worker_MinSpareThreads }}"
  - MaxSpareThreads: "{{ apache_config_mpm_worker_MaxSpareThreads }}"
  - ThreadsPerChild: "{{ apache_config_mpm_worker_ThreadsPerChild }}"
  - MaxRequestsPerChild: "{{ apache_config_mpm_worker_MaxRequestsPerChild }}"

apache_config_mpm_worker__custom: []

apache_config_mpm_worker:
  - name: IfModule
    param: worker.c
    content:
      - options: "{{
          apache_config_mpm_worker__default +
          apache_config_mpm_worker__custom
        }}"

apache_config_mpm:
  - sections: "{{
      apache_config_mpm_prefork +
      apache_config_mpm_worker
    }}"


# Modules
apache_config_modules__default:
  - LoadModule:
    - auth_basic_module
    - modules/mod_auth_basic.so
  - LoadModule:
    - auth_digest_module
    - modules/mod_auth_digest.so
  - LoadModule:
    - authn_file_module
    - modules/mod_authn_file.so
  - LoadModule:
    - authn_alias_module
    - modules/mod_authn_alias.so
  - LoadModule:
    - authn_anon_module
    - modules/mod_authn_anon.so
  - LoadModule:
    - authn_dbm_module
    - modules/mod_authn_dbm.so
  - LoadModule:
    - authn_default_module
    - modules/mod_authn_default.so
  - LoadModule:
    - authz_host_module
    - modules/mod_authz_host.so
  - LoadModule:
    - authz_user_module
    - modules/mod_authz_user.so
  - LoadModule:
    - authz_owner_module
    - modules/mod_authz_owner.so
  - LoadModule:
    - authz_groupfile_module
    - modules/mod_authz_groupfile.so
  - LoadModule:
    - authz_dbm_module
    - modules/mod_authz_dbm.so
  - LoadModule:
    - authz_default_module
    - modules/mod_authz_default.so
  - LoadModule:
    - ldap_module
    - modules/mod_ldap.so
  - LoadModule:
    - authnz_ldap_module
    - modules/mod_authnz_ldap.so
  - LoadModule:
    - include_module
    - modules/mod_include.so
  - LoadModule:
    - log_config_module
    - modules/mod_log_config.so
  - LoadModule:
    - logio_module
    - modules/mod_logio.so
  - LoadModule:
    - env_module
    - modules/mod_env.so
  - LoadModule:
    - ext_filter_module
    - modules/mod_ext_filter.so
  - LoadModule:
    - mime_magic_module
    - modules/mod_mime_magic.so
  - LoadModule:
    - expires_module
    - modules/mod_expires.so
  - LoadModule:
    - deflate_module
    - modules/mod_deflate.so
  - LoadModule:
    - headers_module
    - modules/mod_headers.so
  - LoadModule:
    - usertrack_module
    - modules/mod_usertrack.so
  - LoadModule:
    - setenvif_module
    - modules/mod_setenvif.so
  - LoadModule:
    - mime_module
    - modules/mod_mime.so
  - LoadModule:
    - dav_module
    - modules/mod_dav.so
  - LoadModule:
    - status_module
    - modules/mod_status.so
  - LoadModule:
    - autoindex_module
    - modules/mod_autoindex.so
  - LoadModule:
    - info_module
    - modules/mod_info.so
  - LoadModule:
    - dav_fs_module
    - modules/mod_dav_fs.so
  - LoadModule:
    - vhost_alias_module
    - modules/mod_vhost_alias.so
  - LoadModule:
    - negotiation_module
    - modules/mod_negotiation.so
  - LoadModule:
    - dir_module
    - modules/mod_dir.so
  - LoadModule:
    - actions_module
    - modules/mod_actions.so
  - LoadModule:
    - speling_module
    - modules/mod_speling.so
  - LoadModule:
    - userdir_module
    - modules/mod_userdir.so
  - LoadModule:
    - alias_module
    - modules/mod_alias.so
  - LoadModule:
    - substitute_module
    - modules/mod_substitute.so
  - LoadModule:
    - rewrite_module
    - modules/mod_rewrite.so
  - LoadModule:
    - proxy_module
    - modules/mod_proxy.so
  - LoadModule:
    - proxy_balancer_module
    - modules/mod_proxy_balancer.so
  - LoadModule:
    - proxy_ftp_module
    - modules/mod_proxy_ftp.so
  - LoadModule:
    - proxy_http_module
    - modules/mod_proxy_http.so
  - LoadModule:
    - proxy_ajp_module
    - modules/mod_proxy_ajp.so
  - LoadModule:
    - proxy_connect_module
    - modules/mod_proxy_connect.so
  - LoadModule:
    - cache_module
    - modules/mod_cache.so
  - LoadModule:
    - suexec_module
    - modules/mod_suexec.so
  - LoadModule:
    - disk_cache_module
    - modules/mod_disk_cache.so
  - LoadModule:
    - cgi_module
    - modules/mod_cgi.so
  - LoadModule:
    - version_module
    - modules/mod_version.so

apache_config_modules__custom: []

apache_config_modules:
  - options: "{{
      apache_config_modules__default +
      apache_config_modules__custom
    }}"


# Base
apache_config_base_Include: conf.d/*.conf
apache_config_base_User: apache
apache_config_base_Group: apache
apache_config_base_ServerAdmin: root@localhost
apache_config_base_ServerName: localhost
apache_config_base_UseCanonicalName: "Off"

apache_config_base__default:
  - Include: "{{ apache_config_base_Include }}"
  - User: "{{ apache_config_base_User }}"
  - Group: "{{ apache_config_base_Group }}"
  - ServerAdmin: "{{ apache_config_base_ServerAdmin }}"
  - ServerName: "{{ apache_config_base_ServerName }}"
  - UseCanonicalName: "{{ apache_config_base_UseCanonicalName }}"

apache_config_base__common: []

apache_config_base:
  - options: "{{
      apache_config_base__default +
      apache_config_base__common
    }}"


# Document
apache_config_doc_DocumentRoot: /var/www/html
apache_config_doc_DocumentRoot__options:
  - options:
    - DocumentRoot: "{{ apache_config_doc_DocumentRoot }}"

apache_config_doc_Directory_root_options__default:
  - Options: FollowSymLinks
  - AllowOverride: None

apache_config_doc_Directory_root_options__custom: []

apache_config_doc_Directory_root:
  - name: Directory
    param: /
    content:
      - options: "{{
          apache_config_doc_Directory_root_options__default + 
          apache_config_doc_Directory_root_options__custom
        }}"

apache_config_doc_Directory_html_options__default:
  - Options:
    - Indexes
    - FollowSymLinks
  - AllowOverride: None
  - Order: allow,deny
  - Allow:
    - from
    - all

apache_config_doc_Directory_html_options__custom: []

apache_config_doc_Directory_html:
  - name: Directory
    param: "{{ apache_config_doc_DocumentRoot }}"
    content:
      - options: "{{
          apache_config_doc_Directory_html_options__default +
          apache_config_doc_Directory_html_options__custom
        }}"

apache_config_doc_Directory_UserDir: disabled

apache_config_doc_Directory_userdir:
  - name: IfModule
    param: mod_userdir.c
    content:
      - options:
        - UserDir: "{{ apache_config_doc_Directory_UserDir }}"

apache_config_doc_Directory__sections:
  - sections: "{{
      apache_config_doc_Directory_root +
      apache_config_doc_Directory_html +
      apache_config_doc_Directory_userdir
    }}"

apache_config_doc_Directory_DirectoryIndex:
  - DirectoryIndex:
    - index.html
    - index.html.var

apache_config_doc_Directory_AccessFileName: .htaccess

apache_config_doc_Directory_accessfilename:
  - AccessFileName: "{{ apache_config_doc_Directory_AccessFileName }}"

apache_config_doc_Directory__options:
  - options: "{{
      apache_config_doc_Directory_DirectoryIndex +
      apache_config_doc_Directory_accessfilename
    }}"

apache_config_doc_Files:
  - sections:
    - name: Files
      operator: "~"
      param: "^\\.ht"
      content:
        - options:
          - Order: allow,deny
          - Deny:
            - from
            - all
          - Satisfy:
            - All

apache_config_doc__default: "{{
  apache_config_doc_DocumentRoot__options +
  apache_config_doc_Directory__sections +
  apache_config_doc_Directory__options +
  apache_config_doc_Files
}}"

apache_config_doc__custom: []

apache_config_doc: "{{
  apache_config_doc__default +
  apache_config_doc__custom
}}"


# MIME types
apache_config_mime_TypesConfig: /etc/mime.types
apache_config_mime_DefaultType: text/plain
apache_config_mime_MIMEMagicFile: conf/magic

apache_config_mime:
  - options:
    - TypesConfig: "{{ apache_config_mime_TypesConfig }}"
    - DefaultType: "{{ apache_config_mime_DefaultType }}"
  - sections:
    - name: IfModule
      param: mod_mime_magic.c
      content:
        - options:
          - MIMEMagicFile: "{{ apache_config_mime_MIMEMagicFile }}"


# Logging
apache_config_log_HostnameLookups: "Off"
apache_config_log_ErrorLog: logs/error_log
apache_config_log_LogLevel: warn
apache_config_log_LogFormat_combined: "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\""
apache_config_log_LogFormat_common: "%h %l %u %t \"%r\" %>s %b"
apache_config_log_LogFormat_referer: "%{Referer}i -> %U"
apache_config_log_LogFormat_agent: "%{User-agent}i"
apache_config_log_CustomLog_path: logs/access_log
apache_config_log_CustomLog_type: combined
apache_config_log_ServerSignature: "On"

apache_config_log_options1:
  - HostnameLookups: "{{ apache_config_log_HostnameLookups }}"
  - ErrorLog: "{{ apache_config_log_ErrorLog }}"
  - LogLevel: "{{ apache_config_log_LogLevel }}"

apache_config_log_LogFormat__default:
  - LogFormat:
    - "{{ apache_config_log_LogFormat_combined }}"
    - combined
  - LogFormat:
    - "{{ apache_config_log_LogFormat_common }}"
    - common
  - LogFormat:
    - "{{ apache_config_log_LogFormat_referer }}"
    - referer
  - LogFormat:
    - "{{ apache_config_log_LogFormat_agent }}"
    - agent

apache_config_log_LogFormat__custom: []

apache_config_log_options2:
  - CustomLog:
    - "{{ apache_config_log_CustomLog_path }}"
    - "{{ apache_config_log_CustomLog_type }}"
  - ServerSignature: "{{ apache_config_log_ServerSignature }}"

apache_config_log__default:
    - options: "{{
        apache_config_log_options1 +
        apache_config_log_LogFormat__default +
        apache_config_log_LogFormat__custom +
        apache_config_log_options2
    }}"

apache_config_log__custom: []

apache_config_log: "{{
  apache_config_log__default +
  apache_config_log__custom
}}"


# Icons
apache_config_icons_alias_url: /icons/
apache_config_icons_alias_path: /var/www/icons

apache_config_icons_Directory_options:
  - Options:
    - Indexes
    - MultiViews
    - FollowSymLinks
  - AllowOverride: None
  - Order: allow,deny
  - Allow:
    - from
    - all

apache_config_icons:
  - options:
    - Alias:
      - "{{ apache_config_icons_alias_url }}"
      - "{{ apache_config_icons_alias_path + '/' }}"
  - sections:
    - name: Directory
      param: "{{ apache_config_icons_alias_path }}"
      content:
        - options: "{{
            apache_config_icons_Directory_options
          }}"


# DAV
apache_config_davfs_DAVLockDB: /var/lib/dav/lockdb

apache_config_davfs:
  - sections:
    - name: IfModule
      param: mod_dav_fs.c
      content:
        - options:
          - DAVLockDB: "{{ apache_config_davfs_DAVLockDB }}"


# CGI
apache_config_cgi_ScriptAlias_url: /cgi-bin/
apache_config_cgi_ScriptAlias_path: /var/www/cgi-bin
apache_config_cgi_Directory_options:
  - AllowOverride: None
  - Options: None
  - Order: allow,deny
  - Allow:
    - from
    - all

apache_config_cgi:
  - options:
    - ScriptAlias:
      - "{{ apache_config_cgi_ScriptAlias_url }}"
      - "{{ apache_config_cgi_ScriptAlias_path + '/' }}"
  - sections:
    - name: Directory
      param: "{{ apache_config_cgi_ScriptAlias_path }}"
      content:
        - options: "{{
            apache_config_cgi_Directory_options
          }}"


# Index
apache_config_index_IndexOptions__default:
  - FancyIndexing
  - VersionSort
  - NameWidth=*
  - HTMLTable
  - Charset=UTF-8

apache_config_index_IndexOptions__custom: []

apache_config_index_AddIconByEncoding__default:
  - (CMP,/icons/compressed.gif)
  - x-compress
  - x-gzip

apache_config_index_AddIconByEncoding__custom: []

apache_config_index_AddIconByType__default:
  - AddIconByType:
    - "(TXT,{{ apache_config_icons_alias_url }}text.gif)"
    - text/*
  - AddIconByType:
    - (IMG,{{ apache_config_icons_alias_url }}image2.gif)
    - image/*
  - AddIconByType:
    - (SND,{{ apache_config_icons_alias_url }}sound2.gif)
    - audio/*
  - AddIconByType:
    - (VID,{{ apache_config_icons_alias_url }}movie.gif)
    - video/*
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}binary.gif"
    - .bin
    - .exe
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}binhex.gif"
    - .hqx
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}tar.gif"
    - .tar
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}world2.gif"
    - .wrl
    - .wrl.gz
    - .vrml
    - .vrm
    - .iv
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}compressed.gif"
    - .Z
    - .z
    - .tgz
    - .gz
    - .zip
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}a.gif"
    - .ps
    - .ai
    - .eps
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}layout.gif"
    - .html
    - .shtml
    - .htm
    - .pdf
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}text.gif"
    - .txt
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}c.gif"
    - .c
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}p.gif"
    - .pl
    - .py
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}f.gif"
    - .for
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}dvi.gif"
    - .dvi
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}uuencoded.gif"
    - .uu
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}script.gif"
    - .conf
    - .sh
    - .shar
    - .csh
    - .ksh
    - .tcl
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}tex.gif"
    - .tex
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}bomb.gif"
    - /core
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}back.gif"
    - ..
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}hand.right.gif"
    - README
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}folder.gif"
    - ^^DIRECTORY^^
  - AddIcon:
    - "{{ apache_config_icons_alias_url }}blank.gif"
    - ^^BLANKICON^^

apache_config_index_AddIconByType__custom: []

apache_config_index_DefaultIcon: /icons/unknown.gif
apache_config_index_ReadmeName: README.html
apache_config_index_HeaderName: HEADER.html

apache_config_index_IndexIgnore__default:
  - .??*
  - "*~"
  - "*#"
  - HEADER*
  - README*
  - RCS
  - CVS
  - "*,v"
  - "*,t"

apache_config_index_IndexIgnore__custom: []

apache_config_index:
  - options:
    - IndexOptions: "{{
        apache_config_index_IndexOptions__default +
        apache_config_index_IndexOptions__custom
      }}"
  - options:
    - AddIconByEncoding: "{{
        apache_config_index_AddIconByEncoding__default +
        apache_config_index_AddIconByEncoding__custom
      }}"
  - options: "{{
      apache_config_index_AddIconByType__default +
      apache_config_index_AddIconByType__custom
    }}"
  - options:
    - DefaultIcon: "{{ apache_config_index_DefaultIcon }}"
    - ReadmeName: "{{ apache_config_index_ReadmeName }}"
    - HeaderName: "{{ apache_config_index_HeaderName }}"
    - IndexIgnore: "{{
        apache_config_index_IndexIgnore__default +
        apache_config_index_IndexIgnore__custom
      }}"


# Languages
apache_config_languages_AddLanguage__default:
  - AddLanguage:
    - ca
    - .ca
  - AddLanguage:
    - cs
    - .cz
    - .cs
  - AddLanguage:
    - da
    - .dk
  - AddLanguage:
    - de
    - .de
  - AddLanguage:
    - el
    - .el
  - AddLanguage:
    - en
    - .en
  - AddLanguage:
    - eo
    - .eo
  - AddLanguage:
    - es
    - .es
  - AddLanguage:
    - et
    - .et
  - AddLanguage:
    - fr
    - .fr
  - AddLanguage:
    - he
    - .he
  - AddLanguage:
    - hr
    - .hr
  - AddLanguage:
    - it
    - .it
  - AddLanguage:
    - ja
    - .ja
  - AddLanguage:
    - ko
    - .ko
  - AddLanguage:
    - ltz
    - .ltz
  - AddLanguage:
    - nl
    - .nl
  - AddLanguage:
    - nn
    - .nn
  - AddLanguage:
    - "no"
    - .no
  - AddLanguage:
    - pl
    - .po
  - AddLanguage:
    - pt
    - .pt
  - AddLanguage:
    - pt-BR
    - .pt-br
  - AddLanguage:
    - ru
    - .ru
  - AddLanguage:
    - sv
    - .sv
  - AddLanguage:
    - zh-CN
    - .zh-cn
  - AddLanguage:
    - zh-TW
    - .zh-tw

apache_config_languages_AddLanguage__custom: []

apache_config_languages_LanguagePriority__default:
  - en
  - ca
  - cs
  - da
  - de
  - el
  - eo
  - es
  - et
  - fr
  - he
  - hr
  - it
  - ja
  - ko
  - ltz
  - nl
  - nn
  - "no"
  - pl
  - pt
  - pt-BR
  - ru
  - sv
  - zh-CN
  - zh-TW

apache_config_languages_LanguagePriority__custom: []

apache_config_languages_ForceLanguagePriority:
  - Prefer
  - Fallback

apache_config_languages:
  - options: "{{
      apache_config_languages_AddLanguage__default +
      apache_config_languages_AddLanguage__custom
    }}"
  - options:
    - LanguagePriority: "{{
        apache_config_languages_LanguagePriority__default +
        apache_config_languages_LanguagePriority__custom
      }}"
    - ForceLanguagePriority: "{{
        apache_config_languages_ForceLanguagePriority
      }}"


# Type
apache_config_type_AddType__default:
  - AddType:
    - application/x-compress
    - .Z
  - AddType:
    - application/x-gzip
    - .gz
    - .tgz
  - AddType:
    - application/x-x509-ca-cert
    - .crt
  - AddType:
    - application/x-pkcs7-crl
    - .crl
  - AddHandler:
    - type-map
    - var
  - AddType:
    - text/html
    - .shtml

apache_config_type_AddType__custom: []

apache_config_type_AddDefaultCharset: UTF-8

apache_config_type_AddOutputFilter:
  - INCLUDES
  - .shtml

apache_config_type:
  - options: "{{
      apache_config_type_AddType__default +
      apache_config_type_AddType__custom
    }}"
  - options:
    - AddDefaultCharset: "{{ apache_config_type_AddDefaultCharset }}"
    - AddOutputFilter: "{{
        apache_config_type_AddOutputFilter
      }}"


# Error
apache_config_error_Alias_url: /error/
apache_config_error_Alias_path: /var/www/error
apache_config_error_Directory_AllowOverride: None
apache_config_error_Directory_Options: IncludesNoExec

apache_config_error_Directory_LanguagePriority:
  - en
  - es
  - de
  - fr

apache_config_error_Directory_options__default:
  - AllowOverride: "{{ apache_config_error_Directory_AllowOverride }}"
  - Options: "{{ apache_config_error_Directory_Options }}"
  - AddOutputFilter:
    - Includes
    - html
  - AddHandler:
    - type-map
    - var
  - Order: allow,deny
  - Allow:
    - from
    - all
  - LanguagePriority: "{{ apache_config_error_Directory_LanguagePriority }}"
  - ForceLanguagePriority:
    - Prefer
    - Fallback

apache_config_error_Directory_options__custom: []

apache_config_error:
  - options:
    - Alias:
      - "{{ apache_config_error_Alias_url }}"
      - "{{ apache_config_error_Alias_path + '/' }}"
  - sections:
    - name: IfModule
      param: mod_negotiation.c
      content:
        - sections:
          - name: IfModule
            param: mod_include.c
            content:
              - sections:
                - name: Directory
                  param: "{{ apache_config_error_Alias_path }}"
                  content:
                    - options: "{{
                        apache_config_error_Directory_options__default +
                        apache_config_error_Directory_options__custom
                      }}"


# Browser
apache_config_browser_BrowserMatch__default:
  - BrowserMatch:
    - Mozilla/2
    - nokeepalive
  - BrowserMatch:
    - MSIE 4\.0b2;
    - nokeepalive
    - downgrade-1.0
    - force-response-1.0
  - BrowserMatch:
    - RealPlayer 4\.0
    - force-response-1.0
  - BrowserMatch:
    - Java/1\.0
    - force-response-1.0
  - BrowserMatch:
    - JDK/1\.0
    - force-response-1.0
  - BrowserMatch:
    - Microsoft Data Access Internet Publishing Provider
    - redirect-carefully
  - BrowserMatch:
    - MS FrontPage
    - redirect-carefully
  - BrowserMatch:
    - ^WebDrive
    - redirect-carefully
  - BrowserMatch:
    - ^WebDAVFS/1.[0123]
    - redirect-carefully
  - BrowserMatch:
    - ^gnome-vfs/1.0
    - redirect-carefully
  - BrowserMatch:
    - ^XML Spy
    - redirect-carefully
  - BrowserMatch:
    - ^Dreamweaver-WebDAV-SCM1
    - redirect-carefully

apache_config_browser_BrowserMatch__custom: []

apache_config_browser:
  - options: "{{
      apache_config_browser_BrowserMatch__default +
      apache_config_browser_BrowserMatch__custom
    }}"


# Custom
apache_config__custom: []


# Final http.conf file content
apache_config:
  content: "{{
    apache_config_core +
    apache_config_mpm +
    apache_config_modules +
    apache_config_base +
    apache_config_doc +
    apache_config_mime +
    apache_config_log +
    apache_config_icons +
    apache_config_davfs +
    apache_config_cgi +
    apache_config_index +
    apache_config_languages +
    apache_config_type +
    apache_config_error +
    apache_config_browser +
    apache_config__custom
  }}"


# Path to the vhost files
apache_vhost_path: /etc/httpd/conf.d

# Manage the vhost directory (delete unmanaged vhosts)
apache_vhost_managed: True

# Default vhost
apache_vhost_config__default:
  welcome:
    content:
      - sections:
        - name: LocationMatch
          param: "^/+$"
          content:
            - options:
              - Options: -Indexes
              - ErrorDocument:
                - 403
                - /error/noindex.html

# Custom vhost
apache_vhost_config__custom: {}

# Final vhosts (all files)
apache_vhost_config__tmp: {}

apache_vhost_config: "{{
  apache_vhost_config__tmp.update(apache_vhost_config__default) }}{{
  apache_vhost_config__tmp.update(apache_vhost_config__custom) }}{{
  apache_vhost_config__tmp }}"
```


Dependencies
------------

- [Config Encoders](https://github.com/jtyr/ansible/blob/jtyr-config_encoders/lib/ansible/plugins/filter/config_encoders.py)


License
-------

MIT


Author
------

Jiri Tyr
