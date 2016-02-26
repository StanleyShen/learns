# Public IP vs Private IP
[https://www.iplocation.net/public-vs-private-ip-address](https://www.iplocation.net/public-vs-private-ip-address)

From this page, we can see the ranges of private IPs.


Class			
A	10.0.0.0	10.255.255.255	16,777,216
B	172.16.0.0	172.31.255.255	1,048,576
C	192.168.0.0

| Class| Starting IP Address | Ending IP Address | # of Hosts |
| -----| ------------------- |-------------------|------------|
|   A  |   10.0.0.0			 |  10.255.255.255	 | 16,777,216 |
|   B  |   172.16.0.0		 |  172.31.255.255	 | 1,048,576  |
|   C  |   192.168.0.0		 |  192.168.255.255	 | 65,536	  |

In some case we need to check whether a IP is private one or not
```
var start_ip_address = [ 10 * Math.pow(255, 3),
                        172 * Math.pow(255, 3) + 16 * Math.pow(255, 2),
                        192 * Math.pow(255, 3) + 168 * Math.pow(255, 2) ];
var end_ip_address = [ 10 * Math.pow(255, 3) + 255 * Math.pow(255, 2) + 255 * 255 + 255,
                      172 * Math.pow(255, 3) + 31 * Math.pow(255, 2) + 255 * 255 + 255,
                      192 * Math.pow(255, 3) + 168 * Math.pow(255, 2) + 255 * 255 + 255 ];
                        
function get_public_ip_address(ip_list_string) {
  if (!ip_list_string) {
    return null
  }

  var valid_external_ip_arr = [];

  ip_lists = ip_list_string.replace(/\s+/g, "").split(",");
  for(var i = 0; i < ip_lists.length; ++i) {
    var ip_sections = ip_lists[i].split(".");
    //valid IP is always xx.xx.xx.xx, which has 4 sections
    if (ip_sections.length != 4) {
      continue;
    }

    /* parse all section to a value, if there are parse error in each section, then
       it's not a valid IP address
    */
    var ip_value = 0;
    var is_valid_ip = true;
    for( var j = 0; j < 4; ++ j) {
      if ( ip_sections[j].match(/^\d{0,3}$/) ) {
         try {
          var temp_value = parseInt(ip_sections[j]);
          if ( temp_value > 255) { break; }
           ip_value +=  temp_value * Math.pow(255, 3-j);
         } catch(e) {
            console.log(ip_lists[i] + " is not a valid IP address.");
            is_valid_ip = false;
            break;
         }
      } else {
        is_valid_ip = false;
        break;
      }
    }

    if ( !is_valid_ip ) {
      continue;
    }

    // if the calculated value is in private IP list, then just ignore it
    for( var j = 0; j < start_ip_address.length; j ++) {
      if ( ip_value >= start_ip_address[j] && ip_value <= end_ip_address[j] ) {
        break;
      }
    }

    if ( j == start_ip_address.length ) {
       valid_external_ip_arr.push(ip_lists[i]);
    } else {
      continue;
    }
  }
```