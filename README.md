# icinga2-matrix_notification
Sent Icinga2 notifications to Matrix.org or own Matrix homeserver chat room

* https://matrix.org
> An open network for secure, decentralized communication.

The scripts itself are just a bad clone from the original mail notifications, which are from @sysadmama if I remember correct. The "only" difference is, that instead of creating a mail, we use curl to submit the notification into a room.

To send notifications from Icinga2 into a room, the following conditions are required:
* A matrix compatible server, e.g [matrix-synapse](https://github.com/matrix-org/synapse) or use https://matrix.org
* An access token to get access to the Matrix server
* A Matrix room ID
* Access to the room (invite) as the Matrix user, which sends the notifications.

There exists several ways, but an easy way is to create a new user (for example "monitoring" via login in [Riot web](https://riot.im/app/)); get the access token and invite the new Matrix user into the room, which may was created for the monitoring. A good approach is, to have several rooms, for example devops, sysops, web, dba .... and use the apply rule to assign the correct room with the $notification_matrix_room_id$ .

The following configuration should be work in most cases :-)

## commands.cfg

 * **Host Matrix NotificationCommand**

```
object NotificationCommand "Host Alarm by Matrix" {
    import "plugin-notification-command"
    command = [ SysconfDir + "/icinga2/scripts/matrix-host-notification.sh" ]
    arguments += {
        "-4" = "$notification_address$"
        "-6" = "$notification_address6$"
        "-b" = "$notification_author$"
        "-c" = "$notification_comment$"
        "-d" = {
            required = true
            value = "$notification_date$"
        }
        "-i" = "$notification_icingaweb2url$"
        "-l" = {
            required = true
            value = "$notification_hostname$"
        }
        "-m" = {
            required = true
            value = "$notification_matrix_room_id$"
        }
        "-n" = {
            required = true
            value = "$notification_hostdisplayname$"
        }
        "-o" = {
            required = true
            value = "$notification_hostoutput$"
        }
        "-s" = {
            required = true
            value = "$notification_hoststate$"
        }
        "-t" = {
            required = true
            value = "$notification_type$"
        }
        "-x" = {
            required = true
            value = "$notification_matrix_server$"
        }
        "-y" = {
            required = true
            value = "$notification_matrix_token$"
        }
    }
    vars.notification_address = "$address$"
    vars.notification_address6 = "$address6$"
    vars.notification_author = "$notification.author$"
    vars.notification_comment = "$notification.comment$"
    vars.notification_date = "$icinga.long_date_time$"
    vars.notification_hostdisplayname = "$host.display_name$"
    vars.notification_hostname = "$host.name$"
    vars.notification_hostoutput = "$host.output$"
    vars.notification_hoststate = "$host.state$"
    vars.notification_type = "$notification.type$"
}
```
 * **Service Matrix NotificationCommand**

```
object NotificationCommand "Service Alarm by Matrix" {
    import "plugin-notification-command"
    command = [ SysconfDir + "/icinga2/scripts/matrix-service-notification.sh" ]
    arguments += {
        "-4" = {
            required = true
            value = "$notification_address$"
        }
        "-6" = "$notification_address6$"
        "-b" = "$notification_author$"
        "-c" = "$notification_comment$"
        "-d" = {
            required = true
            value = "$notification_date$"
        }
        "-e" = {
            required = true
            value = "$notification_servicename$"
        }
        "-i" = "$notification_icingaweb2url$"
        "-l" = {
            required = true
            value = "$notification_hostname$"
        }
        "-m" = {
            required = true
            value = "$notification_matrix_room_id$"
        }
        "-n" = {
            required = true
            value = "$notification_hostdisplayname$"
        }
        "-o" = {
            required = true
            value = "$notification_serviceoutput$"
        }
        "-s" = {
            required = true
            value = "$notification_servicestate$"
        }
        "-t" = {
            required = true
            value = "$notification_type$"
        }
        "-u" = {
            required = true
            value = "$notification_servicedisplayname$"
        }
        "-x" = {
            required = true
            value = "$notification_matrix_server$"
        }
        "-y" = {
            required = true
            value = "$notification_matrix_token$"
        }
    }
    vars.notification_address = "$address$"
    vars.notification_address6 = "$address6$"
    vars.notification_author = "$notification.author$"
    vars.notification_comment = "$notification.comment$"
    vars.notification_date = "$icinga.long_date_time$"
    vars.notification_hostdisplayname = "$host.display_name$"
    vars.notification_hostname = "$host.name$"
    vars.notification_servicedisplayname = "$service.display_name$"
    vars.notification_serviceoutput = "$service.output$"
    vars.notification_servicestate = "$service.state$"
    vars.notification_type = "$notification.type$"
```

## services.cfg

```
/**
 * Example Matrix.org apply rules.
 * The "!<id>:matrix.org" needs to be replaced with the room ID
 * for example "!SDFfskjfdszhdaslasdkjhdasd:matrix.org".
 * Also a Matrix access token is required too.
 */

apply Notification "Matrix host problems" to Host {
    import "matrix-host-notification"

    user_groups = host.vars.notification.matrix.groups
    users = host.vars.notification.matrix.users
    vars.notification_matrix_server = "https://matrix.org"
    vars.notification_matrix_room_id = "!<id>:matrix.org"
   // vars.notification_matrix_token = "<access_token>"
    assign where host.vars.notification.matrix
}

apply Notification "Matrix service problems" to Service {
    import "matrix-service-notification"

    user_groups = host.vars.notification.matrix.groups
    users = host.vars.notification.matrix.users
    vars.notification_matrix_server = "https://matrix.org"
    vars.notification_matrix_room_id = "!<id>:matrix.org"
   // vars.notification_matrix_token = "<access_token>"
    assign where host.vars.notification.matrix
}
```

## templates.cfg

```
/**
 * Provides default settings for Matrix.org service notifications.
 */

template Notification "matrix-host-notification" {
  command = "matrix-host-notification"

  states = [ Up, Down ]
  types = [ Problem, Acknowledgement, Recovery, Custom,
            FlappingStart, FlappingEnd,
            DowntimeStart, DowntimeEnd, DowntimeRemoved ]
  vars += {
    // notification_icingaweb2url = "https://www.example.com/icingaweb2"
    notification_logtosyslog = false
  }
    // interval = 0s
    period = "24x7"
}

template Notification "matrix-service-notification" {
  command = "matrix-service-notification"

  states = [ OK, Warning, Critical, Unknown ]
  types = [ Problem, Acknowledgement, Recovery, Custom,
            FlappingStart, FlappingEnd,
            DowntimeStart, DowntimeEnd, DowntimeRemoved ]

  vars += {
    // notification_icingaweb2url = "https://www.example.com/icingaweb2"
    notification_logtosyslog = false
  }
    // interval = 0s
    period = "24x7"
}
```

Just put the both scripts into the /etc/icinga2/scripts/ folder, make them 0755 executable and try it out.
