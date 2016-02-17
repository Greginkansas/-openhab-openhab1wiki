### Zoneminder

I appreciate Zoneminder has the Zoneminder.pm api.  I attempted to use it but with most things zoneminder I found the documentation totally out of date and zero comments in the code.  So instead I achieved my goal by accessing the zoneminder mySQL database direct.  It works rather well.

This is a simple php script that can be executed from a linux commandline if you have the PHP packages installed.  I simply run it under "screen" at boot time with the command "screen -d -m /opt/openhab/scripts/zoneminderAlarm.php".

It serves two purposes:

1) Sets a switch in openhab when motion is recorded on my front drive.  Openhab uses the switch to sound a doorbell sound.  I have all sorts of rules governing when to sound the doorbell and when to simply record a missed visitor, how you have that setup is your own call.

2) The script updates openhab number items.  The items store a count for the number of events in the last 24 hours.

READ THE COMMENTS IN THE SOURCE CODE BELOW to set it up.  Hope this helps someone.
```php
    #!/usr/bin/php
    <?php
    /*
    THIS SCRIPT SERVES TWO PURPOSES.  THE FIRST HALF IS USED TO SOUND A DOORBELL WHEN A
    NEW EVENT IS DETECTED. BY DEFAULT ZONEMINDER MONITOR 1 IS USED FOR THIS AS THATS MY
    MONITOR THAT WATCHES THE FRONT DRIVE.
    
    **** CHECK THE SWITCH STATEMENT IN THE "CHECK FOR EVENTS" SECTION BELOW, BE SURE TO
    USE THE RIGHT MONITOR ID (SEE THE COMMENTS) ****
    
    THE SECOND HALF OF THE SCRIPT HARVESTS THE NUMBER OF EVENTS IN THE LAST 24 HOURS FOR
    ANY NUMBER OF ZONEMINDER MONITORS AND PASSES THE COUNTS TO OPENHAB
    */
    
    //THIS ARRAY CONTAINS THE MONITOR IDS AND THE OPENHAB ITEMS YOU WANT TO UPDATE.  THE OPENHAB ITEM IS A SIMPLE NUMBER ITEM THAT STORES THE COUNT
    $countDetails = array(
                       0 => array('monitorId' => '1', 'openhabItem' => 'frontGardenCount', 'lastCount' => 0),
                       1 => array('monitorId' => '2', 'openhabItem' => 'rearGardenCount', 'lastCount' => 0),
                       2 => array('monitorId' => '5', 'openhabItem' => 'rearDoorCount', 'lastCount' => 0),
                    );
    
    function getValByID($id) {
      $res = file_get_contents("http://192.168.1.121:8080/rest/items/" . $id . "/state");
      return $res;
    }
    
    function doPostRequest($item, $data) {
      $url = "http://192.168.1.121:8080/rest/items/" . $item;
    
      $options = array(
        'http' => array(
            'header'  => "Content-type: text/plain\r\n",
            'method'  => 'POST',
            'content' => $data  //http_build_query($data),
        ),
      );
    
      $context  = stream_context_create($options);
      $result = file_get_contents($url, false, $context);
    
      return $result;
    }
    
    $lastId = 0;
    $countCheckTimestamp = time();
    
    //FIRST RUN, BEFORE WE GO INTO THE MAIN LOOP MAKE SURE ALL COUNTS IN OPENHAB ARE SET TO 0
    foreach ($countDetails as $curCount) {
      doPostRequest($curCount['openhabItem'], "0");
    }
    
    $con = mysqli_connect("localhost", "root", "r292nda", "zm");
    
    if (mysqli_connect_errno($con)) {
      exec('logger "zoneminderAlarm cannot connect to database"');
    } else {
      while(1 == 1) {
    
        //CHECK FOR EVENTS
        if ($result = $con->query("select * from Events order by Id desc limit 1")) {
          while ($row = $result->fetch_row()) {
            if ($lastId == 0) {
              $lastId = $row[0];
            }
    
            if ($row[0] > $lastId) {
              $lastId = $row[0];
    
    	  //$row[1] IS THE MONITOR ID IN ZONEMINDER, TO FIND THIS GO TO THE ZONEMINDER WEBSITE, POINT TO THE LINK TO VIEW THE MONITOR, LOOK AT THE LINK URL, MID= IS THE MONITOR ID
              switch ($row[1]) {
                case "1":
                  doPostRequest("doorbellState", "ON");
                  break;
                case "2":
                  doPostRequest("privacyState", "ON");
                  break;
              }
            }
          }
        }
    
        //EVERY 30 SECONDS UPDATE THE EVENT COUNTS
        if (time() - $countCheckTimestamp >= 30) {
          $countCheckTimestamp = time();
          //GET EVENT COUNTS
          $curDate = date("Y-m-d");
    
          foreach ($countDetails as $key => $curCount) {
    
            if ($result = $con->query("select count(Id) as num from Events where MonitorId = '" . $curCount['monitorId'] . "' and StartTime like '" . $curDate . "%'")) {
              $row = $result->fetch_row();
    
              if (is_numeric($row[0])) {
                //ONLY SEND A COUNT TO OPENHAB IF IT HAS CHANGED
                if ($row[0] <> $curCount['lastCount']) {
                  $countDetails[$key]['lastCount'] = $row[0];
                  doPostRequest($curCount['openhabItem'], $row[0]);
                }
              } else {
                $countDetails[$key]['lastCount'] = "0";
                doPostRequest($curCount['openhabItem'], "0");
              }
            }
          } //foreach ($countDetails as $curCount)
        } //if (time() - $countCheckTimestamp > 30)
    
        sleep(2);
      }
    }
    ?>
```

## API (as of zoneminder V1.29)
Starting with version 1.29 zoneminder features an API. Monitor's function (i.e. Modect, Nodect, Mocord, etc.) may be established with a Json post to the server. This allows for simple "arming" of cameras (e.g. when there's no one home). For example, in order to set monitor 1 to modect (record on motion detection) with a rule:

    sendHttpPostRequest("http://IPofZoneminderServer/zm/api/monitors/1.json", "application/x-www-form-urlencoded", "Monitor[Function]=Modect")

To set its function to monitor (no recording, just being able to watch live):

    sendHttpPostRequest("http://IPofZoneminderServer/zm/api/monitors/1.json", "application/x-www-form-urlencoded", "Monitor[Function]=Monitor") 