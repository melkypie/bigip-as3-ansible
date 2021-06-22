##### Global BIGIP config tenant/common declaration
These should be defined in `group_vars/all.yml` or under `group_vars/all/`.

They will be used in every BIGIP AS3 configuration and can be overriden by individual BIGIP AS3 configs.
```yaml
global:
  tenants:
    < tenant_name >:
      apps:
        # This will be used as the app name so it will come out as /tenant_name/APP_NAME
        - < app_name >
  common:
    objects:
      # This name won't be used in final declaration, it is just there for ansible to be able to find the correct variable
      - < custom_as3_object_name >
```

##### Individual BIGIP config tenant/common declaration

```yaml
tenants:
  < tenant_name >:
    apps:
      # This will be used as the app name so it will come out as /tenant_name/APP_NAME
      - < app_name >
common:
  objects:
    # This name won't be used in final declaration, it is just there for ansible to be able to find the correct variable
    - < custom_as3_object_name >
```

##### Object definition
(these are defined under `common.objects` and use close to real AS3 configuration variables).

Supported AS3 objects can be found under [roles/bigip_as3_gen/templates/app/objects](roles/bigip_as3_gen/templates/app/objects) and their reference in [AS3 object schema reference](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/refguide/schema-reference.html) (not all options are supported)

```yaml
< custom_as3_object_name >:
  name: < object_name > # This name will be the one that is actually used
  type: < as3_object >
  # Rest of the configuration is the same as actual AS3 object (not all options are supported)
  ...
```

##### Application definition
(this can be defined under `group_vars/all` or under a specific BIGIP `host_vars/bigip1.yml` or it's folder `host_vars/bigip1/`)

```yaml
< app_name >:
  name: < virtual_server_name | string >
  type: < application_template >
  # Application specific properties follow, see Application templates
  ...
```

##### Application templates
- **https**
```yaml
  persistenceMethods: < none > # This can be aither an array (if using persistenceMethods) or none (when disabling them)
    - < source-address | cookie | ... | default -> cookie>
  # You can either define one of the options or define your own pool (leaving empty the options, otherwise it is not valid YAML)
  snat: < auto | none | self -> default auto >
    name: < string >
    snatAddresses:
      - < IP_address >
  vips:
    # Using multiple will generate multiple virtual server
    - ip: < IP_address >
      source: < IP_address/Mask >
  pool:
    name: < string >
    lbMode: < load_balancing_mode > # See AS3 reference for possible options
    members:
      - addr: < IP_address >
        port: < port | integer >
        monitors:
          - name: < string or bigip_path >
            ... # The configuration is the same as a regular pool monitor (Look below)
    monitors:
      # If using a common object use /Common/Shared/monitor_name with location: common option
      # If creating a new monitor define just monitor_name
      - name: < string or bigip_path >
        # Only if using a common object, otherwise the option should not be set
        location: < common >
        type: < dns | external | ftp | http | https | http2 | icmp | ldap | mysql | postgresql | radius | sip | smtp | tcp | tcp-half-open | udp >
        receive: < receive_string | string >
        send: < send_string | string >
        # In case of non standard port
        port: < port | integer >
        # Used for type: dns
        queryType: < a | aaaa | default -> a >
        # Used for type: dns
        queryname: < hostname_to_query | string >
        # Used for type: icmp
        transparent: < true | false | default -> false >
  tls:
    name: < string > # Clientssl profile name
    # Use one of the cipher groups provided by F5, ex. /Common/f5-secure
    cipherGroup: < cipher_group | bigip_path >
    certs:
      - name: < certificate_name | string >
        key: < private_key | bigip_path >
        cert: < certificate | bigip_path >
        # Optional
        chain: < chainCA | bigip_path >
        # Optional
        passphrase:
          # Depending on type this can be a plaintext base64 encoded passphrase or BIGIP Secure Vault encrypted string encoded in base64
          # See https://support.f5.com/csp/article/K73034260 how to get a BIGIP Secure Vault encrypted string (when you retrieve it also needs to be encoded in base64)
          # Make sure your string does not contain trailing newlines as BIGIP will fail to decode it then (use the -n option with echo which will remove them)
          # echo -n 'certificate1230' | base64 -w 0
          value: < base64_string >
          # Required if defining password (value is based on type (none = plaintext bas64, f5sv = BIGIP Secure Vault encrypted string which is then encoded in base64))
          type: < none | f5sv >
  profile:
    http:
      name: < string or bigip_path >
      # Only use if name is /Common/Shared/http_profile (this should be the only option if defining it, apart from name )
      location: < common >
      viaRequest: < remove | append | preserve >
      viaResponse: < remove | append | preserve >
    tcp:
      name: < string or bigip_path >
      # Only use if name is /Common/Shared/tcp_profile (this should be the only option if defining it, apart from name )
      location: < common >
      autoProxyBufferSize: < boolean | default -> true >
      autoReceiveWindowSize: < boolean | default -> true >
      autoSendBufferSize: < boolean | default -> true >
      minimumRto: < integer[1,5000] | default -> 1000 >
      proxyBufferHigh: < integer[64,33554432] | default -> 262144 >
      proxyBufferLow: < integer[64,33554432] | default -> 196608 >
      receiveWindowSize: < integer[64,33554432] | default -> 131072 >
      sendBufferSize: < integer[64,33554432] | default -> 262144 >
      initRwnd: < integer[0,64] | default -> 16 >
      initCwnd: < integer[0,64] | default -> 16 >
      maxSegmentSize: < integer | default -> 0 >
      nagle: < auto | disable | enable | default -> auto >
      pushFlag: < auto | default | none | one | default -> auto >
      congestionControl: < default -> woodside > # See AS3 reference for list of options
  vlans:
    - < bigip_path >
  iRules:
    - name: < string or bigip_path >
      # If using bigip_path
      location: common
      base64: < base64_encoded_irule | string >
```
- **psql**
```yaml
  # You can either define one of the options or define your own pool (leaving empty the options, otherwise it is not valid YAML)
  snat: < auto | none | self -> default auto >
    name: < string >
    snatAddresses:
      - < IP_address >
  persistenceMethods: < none > # This can be aither an array (if using persistenceMethods) or none (when disabling them)
    - < source-address | cookie | ... | default -> cookie>
  vips:
    # Using multiple will generate multiple virtual server
    - ip: < IP_address >
      source: < IP_address/Mask >
  pool:
    # Defining multiple read/write pools will generate mutliple virtual servers ( for each read/write pool)
    - write:
        name: < string >
        # Port that will be used for creating the virtual server
        port: < integer >
        lbMode: < load_balancing_mode > # See AS3 reference for possible options
        members:
          - addr: < IP_address >
            port: < port | integer >
            monitors:
              - name: < string or bigip_path >
                ... # The configuration is the same as a regular pool monitor (Look below)
        monitors:
          # If using a common object use /Common/Shared/monitor_name with location: common option
          # If creating a new monitor define just monitor_name
          - name: < string or bigip_path >
            # Only if using a common object, otherwise the option should not be set
            location: < common >
            type: < dns | external | ftp | http | https | http2 | icmp | ldap | mysql | postgresql | radius | sip | smtp | tcp | tcp-half-open | udp >
            receive: < receive_string | string >
            send: < send_string | string >
            # In case of non standard port
            port: < port | integer >
            # Used for type: dns
            queryType: < a | aaaa | default -> a >
            # Used for type: dns
            queryname: < hostname_to_query | string >
            # Used for type: icmp
            transparent: < true | false | default -> false >
      read:
        # Same as above but just for read pool
        ...
  profile:
    tcp:
      name: < string or bigip_path >
      # Only use if name is /Common/Shared/tcp_profile (this should be the only option if defining it, apart from name )
      location: < common >
      autoProxyBufferSize: < boolean | default -> true >
      autoReceiveWindowSize: < boolean | default -> true >
      autoSendBufferSize: < boolean | default -> true >
      minimumRto: < integer[1,5000] | default -> 1000 >
      proxyBufferHigh: < integer[64,33554432] | default -> 262144 >
      proxyBufferLow: < integer[64,33554432] | default -> 196608 >
      receiveWindowSize: < integer[64,33554432] | default -> 131072 >
      sendBufferSize: < integer[64,33554432] | default -> 262144 >
      initRwnd: < integer[0,64] | default -> 16 >
      initCwnd: < integer[0,64] | default -> 16 >
      maxSegmentSize: < integer | default -> 0 >
      nagle: < auto | disable | enable | default -> auto >
      pushFlag: < auto | default | none | one | default -> auto >
      congestionControl: < default -> woodside > # See AS3 reference for list of options
  iRules:
    - name: < string or bigip_path >
      # If using bigip_path
      location: common
      base64: < base64_encoded_irule | string >
  vlans:
    - < bigip_path >
```
- **forward**
```yaml
  forwardType: < ip | l2 | default -> ip >
  # This needs to be defined, usually will be 0 for any
  port: < integer >
  # You can either define one of the options or define your own pool (leaving empty the options, otherwise it is not valid YAML)
  snat: < auto | none | self -> default auto >
    name: < string >
    snatAddresses:
      - < IP_address >
  l4: < any | tcp | udp | default -> any >
  vips:
    # Using multiple will generate multiple virtual server
    - ip: < IP_address >
      source: < IP_address/Mask >
  profile:
    l4:
      name: < string or bigip_path >
      # Only use if name is /Common/Shared/udp_profile (this should be the only option if defining it, apart from name )
      location: < common >
      looseClose: < boolean | default -> false >
      looseInit: < boolean | default -> false >
      resetOnTimeout: < boolean | default -> true >
      tcpHandshakeTimeout: < integer[-1,86400] | default -> 5 >
  vlans:
    - < bigip_path >
```
- **dns**
```yaml
  # Creates individual virtual servers for TCP/UDP, every parameter that can be defined in top_level can be defined under these as well
  tcp:
    name: < string >
    ...
  udp:
    name: < string >
    ...
  # You can either define one of the options or define your own pool (leaving empty the options, otherwise it is not valid YAML)
  snat: < auto | none | self -> default auto >
    name: < string >
    snatAddresses:
      - < IP_address >
  persistenceMethods: < none > # This can be aither an array (if using persistenceMethods) or none (when disabling them)
    - < source-address | cookie | ... | default -> cookie>
  vips:
    # Using multiple will generate multiple virtual server
    - ip: < IP_address >
      source: < IP_address/Mask >
  pool:
    name: < string >
    lbMode: < load_balancing_mode > # See AS3 reference for possible options
    members:
      - addr: < IP_address >
        port: < port | integer >
        monitors:
          - name: < string or bigip_path >
            ... # The configuration is the same as a regular pool monitor (Look below)
    monitors:
      # If using a common object use /Common/Shared/monitor_name with location: common option
      # If creating a new monitor define just monitor_name
      - name: < string or bigip_path >
        # Only if using a common object, otherwise the option should not be set
        location: < common >
        type: < dns | external | ftp | http | https | http2 | icmp | ldap | mysql | postgresql | radius | sip | smtp | tcp | tcp-half-open | udp >
        receive: < receive_string | string >
        send: < send_string | string >
        # In case of non standard port
        port: < port | integer >
        # Used for type: dns
        queryType: < a | aaaa | default -> a >
        # Used for type: dns
        queryname: < hostname_to_query | string >
        # Used for type: icmp
        transparent: < true | false | default -> false >
  profile:
    tcp:
      name: < string or bigip_path >
      # Only use if name is /Common/Shared/tcp_profile (this should be the only option if defining it, apart from name )
      location: < common >
      autoProxyBufferSize: < boolean | default -> true >
      autoReceiveWindowSize: < boolean | default -> true >
      autoSendBufferSize: < boolean | default -> true >
      minimumRto: < integer[1,5000] | default -> 1000 >
      proxyBufferHigh: < integer[64,33554432] | default -> 262144 >
      proxyBufferLow: < integer[64,33554432] | default -> 196608 >
      receiveWindowSize: < integer[64,33554432] | default -> 131072 >
      sendBufferSize: < integer[64,33554432] | default -> 262144 >
      initRwnd: < integer[0,64] | default -> 16 >
      initCwnd: < integer[0,64] | default -> 16 >
      maxSegmentSize: < integer | default -> 0 >
      nagle: < auto | disable | enable | default -> auto >
      pushFlag: < auto | default | none | one | default -> auto >
      congestionControl: < default -> woodside > # See AS3 reference for list of options
    udp:
      name: < string or bigip_path >
      # Only use if name is /Common/Shared/udp_profile (this should be the only option if defining it, apart from name )
      location: < common >
      datagramLoadBalancing: < boolean | default -> false >
  iRules:
    - name: < string or bigip_path >
      # If using bigip_path
      location: common
      base64: < base64_encoded_irule | string >
  vlans:
    - < bigip_path >
```