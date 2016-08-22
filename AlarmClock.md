Below you'll find two examples on how to realize an alarm clock with openHAB. Example 1 does only use one item to control the alarm time. Example II will control the hours and minutes seperately.

* [Example I] (AlarmClock#alarm-clock---example-i)
 * [Items] (AlarmClock#items)
 * [Rules] (AlarmClock#rules)
 * [Sitemap] (AlarmClock#sitemap)

* [Example II] (AlarmClock#alarm-clock---example-ii)
 * [Items] (AlarmClock#items-1)
 * [Rules] (AlarmClock#rules-1)
 * [Sitemap] (AlarmClock#sitemap-1)

*  [Example III] (AlarmClock#alarm-clock---example-iii)
 * [Items] (AlarmClock#items-2)
 * [Rules] (AlarmClock#rules-2)
 * [Sitemap] (AlarmClock#sitemap-2)

* [References] (AlarmClock#references)

# Alarm Clock - Example I
There is no 'time' widget and using numbers/dimmers/setpoints; if you do not want to have two items, one for hours and one for minutes this example is for you ...

Basically you are specifying the alarm time as a number from 0-600 mins (i.e. 0-10 hours). The base time is midnight, so this corresponds to a time between midnight and 10am. Obviously very easy to change the Setpoint config in your sitemap to extend or restrict this. I have it set to 'step' in 5 min increments, but you could make this 15 mins to make it easier to quickly set coarse alarm times if you wanted.

Whenever you change the alarm time mins item using the Setpoint widget, the alarm time is calculated and a display item is updated, so you can show the nicely formatted alarm time in the sitemap. This item is not used for anything other than display. 

The reason I have two frames for displaying the alarm details is so I can merge both the alarm switch and time items into one sitemap widget. So if the alarm is enabled I hardcode the presence-on icon and display the alarm time, if it is disabled I hard-code the presence-off icon. By doing this I can display the alarm time and have an icon showing the alarm state. Drilling down into that frame gives you the option to disable the alarm and change the time. [[1](https://groups.google.com/d/msg/openhab/F2hqVwjbRC0/b1s44_MWGygJ)]

## Items
    Switch     Alarm_Master          "Master Alarm"       <presence>       (Alarms)
    Number     Alarm_MasterTimeMins  "Master Alarm"       <clock>          (Alarms)
    String     Alarm_MasterTime      "Master Alarm [%s]"  <clock>
    Switch     Alarm_MasterEvent     "Master Alarm Event" <alarm>          (AlarmEvents)      { autoupdate="false" }

## Rules
```Xtend

	import org.joda.time.*

	var Timer masterAlarmTime = null

	rule "Master bedroom alarm time"
	when
		Time cron "0 5 0 * * ?" or
		Item Alarm_MasterTimeMins received update
	then
		var int minutes = (Alarm_MasterTimeMins.state as DecimalType).intValue()

		if (masterAlarmTime != null)
			masterAlarmTime.cancel()

		// work out when the alarm is to fire - start from midnight
		var DateTime alarmTime = parse(now.getYear() + "-" + now.getMonthOfYear() + "-" + now.getDayOfMonth() + "T00:00")

		// add the number of minutes selected
		alarmTime = alarmTime.plusMinutes(minutes)

		// if we have already past the alarm time then set it for the following day
		if (alarmTime.beforeNow)
			alarmTime = alarmTime.plusDays(1)

		// create a timer to execute the alarm at the specified time
		masterAlarmTime = createTimer(alarmTime) [| 
			if (Alarm_Master.state == ON && Holiday.state == OFF && now.getDayOfWeek() < 6) 
				Alarm_MasterEvent.sendCommand(ON)
		]

		// update the alarm display time    
		Alarm_MasterTime.sendCommand(String::format("%02d:%02d", alarmTime.getHourOfDay(), alarmTime.getMinuteOfHour()))
	end

	rule "Master bedroom alarm"
	when
		Item Alarm_MasterEvent received command ON
	then
		// do your alarm stuff - turn on radio, dim up lights, start the coffee machine...
	end
```
# Sitemap
	Frame label="Alarm" {
		Text item=Alarm_MasterTime icon="presence-on" visibility=[Alarm_Master==ON] {
			Frame label="Master Alarm" {
				Switch item=Alarm_Master
				Text item=Alarm_MasterTime
				Setpoint item=Alarm_MasterTimeMins minValue=0 maxValue=600 step=5
			}
		}
		Text item=Alarm_MasterTime icon="presence-off" visibility=[Alarm_Master==OFF] {
			Frame label="Master Alarm" {
				Switch item=Alarm_Master
				Text item=Alarm_MasterTime
				Setpoint item=Alarm_MasterTimeMins minValue=0 maxValue=600 step=5
			}
		}
	}

# Alarm Clock - Example II

![UI (greenT)](https://dl.dropboxusercontent.com/u/1781347/wiki/2014-12-07%2018_30_22-openHAB.png)

## Items
```
Switch	weckerMontag     "Montag"     <switch>	(gWeckerWochentage)	
Switch	weckerDienstag   "Dienstag"   <switch>	(gWeckerWochentage)	
Switch	weckerMittwoch   "Mittwoch"	  <switch>	(gWeckerWochentage)	
Switch	weckerDonnerstag "Donnerstag" <switch>	(gWeckerWochentage)	
Switch	weckerFreitag    "Freitag"    <switch>	(gWeckerWochentage)	
Switch	weckerSamstag    "Samstag"    <switch>	(gWeckerWochentage)	
Switch	weckerSonntag    "Sonntag"    <switch>	(gWeckerWochentage)	

String weckerZeitMessage "%s"

Number weckerZeitStunde "Stunde [%d]" <clock> (gWeckerZeit)
Number weckerZeitMinute "Minute [%d]" <clock> (gWeckerZeit)
```
## Rules
```Xtend
import org.openhab.core.library.types.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*
import org.openhab.action.squeezebox.*

import java.util.concurrent.locks.ReentrantLock

var Timer timer1 = null
var java.util.concurrent.locks.ReentrantLock lock1 = new java.util.concurrent.locks.ReentrantLock()

rule "Initialization"
 when 
   System started
 then
     postUpdate(weckerZeitStunde,  8)
     postUpdate(weckerZeitMinute, 15)
     postUpdate(weckerMontag,     ON)
     postUpdate(weckerDienstag,   ON)
     postUpdate(weckerMittwoch,   ON)
     postUpdate(weckerDonnerstag, ON)
     postUpdate(weckerFreitag,    ON)
     postUpdate(weckerSamstag,    OFF)
     postUpdate(weckerSonntag,    OFF)
 end

rule "Weckzeit"
when
	Item weckerZeitStunde changed or 
	Item weckerZeitMinute changed
then
  // If the UI to change the Alarm time is clicked several times the code below
  // is subject to race conditions. Therefore we make sure that all events 
  // are processed one after the other.
  lock1.lock()
  try {
    var String msg = ""
    
    // Copy the Alarm-Time from the UI to local variables
    var stunde = weckerZeitStunde.state as DecimalType
    var minute = weckerZeitMinute.state as DecimalType
  
    // Combine the hour and minutes to one string to be displayed in the 
    // user interface
    if (stunde < 10) { msg = "0" } 
    msg = msg + weckerZeitStunde.state.format("%d") + ":"
    
    if (minute < 10) { msg = msg + "0" }
    msg = msg + weckerZeitMinute.state.format("%d")
    postUpdate(weckerZeitMessage,msg)
  
    // calculate the alarm time [min]
    var int weckzeit1
    weckzeit1 = (weckerZeitStunde.state as DecimalType).intValue * 60 + 
                (weckerZeitMinute.state as DecimalType).intValue
    weckzeit1 = weckzeit1.intValue
  
    // calculate current time [min]
    var int jetzt1
    jetzt1 = now.getMinuteOfDay
    jetzt1 = jetzt1.intValue

    // calculate the difference between the requested alarm time and 
    // current time (again in minutes)  
    var int delta1
    delta1 = (weckzeit1 - jetzt1)
    delta1 = delta1.intValue
    
    // add one day (1440 minutes) if alarm time for today already passed
    if (jetzt1 > weckzeit1) { delta1 = delta1 + 1440 }
    
    // check if there is already an alarm timer; cancel it if present
    if (timer1 != null) {
       timer1.cancel
       timer1 = null
    }
    
    // create a new timer using the calculated delta [min]
    timer1 = createTimer(now.plusMinutes(delta1)) [|
        // This code will be executed if the timer triggers
        // ************************************************
        // check if alarm clock is armed for this weekday
        var Number day = now.getDayOfWeek
        if (((day == 1) && (weckerMontag.state == ON))     ||
            ((day == 2) && (weckerDienstag.state == ON))   ||
            ((day == 3) && (weckerMittwoch.state == ON))   ||
            ((day == 4) && (weckerDonnerstag.state == ON)) ||
            ((day == 5) && (weckerFreitag.state == ON))    ||
            ((day == 6) && (weckerSamstag.state == ON))    ||
            ((day == 7) && (weckerSonntag.state == ON))) {
                // The things to do if the alarm clock is enabled for this day of week: 
                // ...
                // ...
           }
           // Re-Arm the timer to trigger again tomorrow (same time) 
           timer1.reschedule(now.plusHours(24))
        // ************************************************
        // Here the code ends that executes once the timer triggers 
        ]
  } finally  {
     // release the lock - we are ready to process the next event
     lock1.unlock()
  }
end
```
## Sitemap

    sitemap alarmclock
    {
        Frame label="System" {
                
            Text label="Wecker [%s]" item=weckerZeitMessage icon="clock" {
                Frame label="Zeit" {
                    Setpoint item=weckerZeitStunde minValue=0 maxValue=23 step=1
                    Setpoint item=weckerZeitMinute minValue=0 maxValue=55 step=5
                }
                Frame label="Wochentage" {
                    Switch item=weckerMontag
                    Switch item=weckerDienstag
                    Switch item=weckerMittwoch
                    Switch item=weckerDonnerstag
                    Switch item=weckerFreitag
                    Switch item=weckerSamstag
                    Switch item=weckerSonntag
                }
            }
        }	
    }

# Alarm Clock - Example III
This example starts from the II one and adds the feature to execute different task after the alarm time with the possibility to change the task offset execution time. So we can set up for instance: play alarm sound, switch on light and switch on heating at 6:30 (alarm time), then after 15 minutes switch on the light in kitchen and switch on the radio, after 20 minutes (from alarm time, so 5 minutes after the previous task) switch off the light in bedroom, after 50 minutes from wake up time switch off light in kitchen and radio.
Each step executes a script (alarm1, alarm2, alarm3, alarm4) where it is possible to write the actions to execute, this logic keep separated the main alarm rules and the actions. 



## Items
```
Group gAlarmWeekdays
Group gAlarmTime

Switch	alarmEnabled	"Enabled"	<switch>	(gAlarmTime)
Switch	radioEnabled	"Radio"		<switch>	(gAlarmTime)

Switch  alarmMonday     "Monday"	<switch>	(gAlarmWeekdays) 
Switch  alarmTuesday   	"Tuesday"	<switch>	(gAlarmWeekdays)
Switch  alarmWensday   	"Wensday"	<switch>	(gAlarmWeekdays)
Switch  alarmThursday 	"Thursday"	<switch>	(gAlarmWeekdays)
Switch  alarmFriday    	"Friday"	<switch>	(gAlarmWeekdays)
Switch  alarmSaturday	"Saturday"	<switch>	(gAlarmWeekdays)
Switch  alarmSunday    	"Sunday "	<switch>	(gAlarmWeekdays)

String alarmTimeMessage "%s"

Number alarmTimeHour 		"Hours [%d]" 		<clock> (gAlarmTime)
Number alarmTimeMinutes 	"Minutes [%d]" 		<clock> (gAlarmTime)
Number alarmOffset1Minutes 	"Offset 1 Minutes [%d]" <clock> (gAlarmTime)
Number alarmOffset2Minutes 	"Offset 2 Minutes [%d]" <clock> (gAlarmTime)
Number alarmOffset3Minutes 	"Offset 3 Minutes [%d]" <clock> (gAlarmTime)
```

## Rules
```
/************** Alarm ***********/

import org.openhab.core.library.types.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*
import org.openhab.action.squeezebox.*

import java.util.concurrent.locks.ReentrantLock

var Timer timer0 = null
var Timer timer1 = null
var Timer timer2 = null
var Timer timer3 = null

var java.util.concurrent.locks.ReentrantLock lock1 = new java.util.concurrent.locks.ReentrantLock()

rule "Alarm"
when
    Item alarmTimeHour changed or
    Item alarmTimeMinutes changed or
    Item alarmOffset1Minutes changed or
    Item alarmOffset2Minutes changed or
    Item alarmOffset3Minutes changed or
then
	if(alarmEnabled.state != Uninitialized && alarmTimeHour.state != Uninitialized && alarmTimeMinutes.state != Uninitialized && alarmOffset1Minutes.state != Uninitialized && alarmOffset2Minutes.state != Uninitialized && alarmOffset3Minutes.state != Uninitialized && alarmTuesday.state != Uninitialized && alarmWensday.state != Uninitialized && alarmThursday.state != Uninitialized && alarmFriday.state != Uninitialized && alarmSaturday.state != Uninitialized && alarmSunday.state != Uninitialized)
	{
	  // If the UI to change the Alarm time is clicked several times the code below
	  // is subject to race conditions. Therefore we make sure that all events 
	  // are processed one after the other.
	  lock1.lock()
	  try {
		var String msg = ""
		
		logInfo("rules","Alarm has been changed.")

		// Copy the Alarm-Time from the UI to local variables
		var hour = alarmTimeHour.state as DecimalType
		var minute = alarmTimeMinutes.state as DecimalType

		// Combine the hour and minutes to one string to be displayed in the 
		// user interface
		if (hour < 10) { msg = "0" } 
		msg = msg + alarmTimeHour.state.format("%d") + ":"

		if (minute < 10) { msg = msg + "0" }
		msg = msg + alarmTimeMinutes.state.format("%d")
		postUpdate(alarmTimeMessage,msg)
		logInfo("rules","New alarm time: "+msg)

		// calculate the alarm time [min]
		var int alarm1
		alarm1 = (alarmTimeHour.state as DecimalType).intValue * 60 + 
					(alarmTimeMinutes.state as DecimalType).intValue
		alarm1 = alarm1.intValue

		// calculate current time [min]
		var int hour1
		hour1 = now.getMinuteOfDay
		hour1 = hour1.intValue

		// calculate the difference between the requested alarm time and 
		// current time (again in minutes)  
		var int delta1
		delta1 = (alarm1 - hour1)
		delta1 = delta1.intValue

		// add one day (1440 minutes) if alarm time for today already passed
		if (hour1 > alarm1) { delta1 = delta1 + 1440 }
		
		// set the others timer based on the offsets
		var int delta2
		delta2 = delta1 + (alarmOffset1Minutes.state as DecimalType).intValue
		
		var int delta3
		delta3 = delta1 + (alarmOffset2Minutes.state as DecimalType).intValue
		
		var int delta4
		delta4 = delta1 + (alarmOffset3Minutes.state as DecimalType).intValue

		// check if there is already an alarm timer0; cancel it if present
		if (timer0 != null) {
		   timer0.cancel
		   timer0 = null
		}

		// create a new timer using the calculated delta [min]
		timer0 = createTimer(now.plusMinutes(delta1)) [|
			// This code will be executed if the timer triggers
			// ************************************************
			// check if alarm clock is enabled and armed for this weekday
			
			// Re-Arm the timer to trigger again tomorrow (same time) 
			timer0.reschedule(now.plusHours(24))
			
			if(alarmEnabled.state == ON)
			{
				var Number day = now.getDayOfWeek
				if (((day == 1) && (alarmMonday.state == ON))	||
					((day == 2) && (alarmTuesday.state == ON))	||
					((day == 3) && (alarmWensday.state == ON))	||
					((day == 4) && (alarmThursday.state == ON))	||
					((day == 5) && (alarmFriday.state == ON))	||
					((day == 6) && (alarmSaturday.state == ON))	||
					((day == 7) && (alarmSunday.state == ON))) {
						// The things to do if the alarm clock is enabled for this day of week: 
						callScript("alarm1")
				}//if
			}//if enabled			
			]
		logInfo("rules","Alarm, timer0 set.")
		// check if there is already an alarm timer1; cancel it if present
		if (timer1 != null) {
		   timer1.cancel
		   timer1 = null
		}
		
		// create a new timer using the calculated delta [min]
		timer1 = createTimer(now.plusMinutes(delta2)) [|
			// This code will be executed if the timer triggers
			// ************************************************
			// check if alarm clock is enabled and armed for this weekday
			
			// Re-Arm the timer to trigger again tomorrow (same time) 
			timer1.reschedule(now.plusHours(24))
			
			if(alarmEnabled.state == ON)
			{
				var Number day = now.getDayOfWeek
				if (((day == 1) && (alarmMonday.state == ON))	||
					((day == 2) && (alarmTuesday.state == ON))	||
					((day == 3) && (alarmWensday.state == ON))	||
					((day == 4) && (alarmThursday.state == ON))	||
					((day == 5) && (alarmFriday.state == ON))	||
					((day == 6) && (alarmSaturday.state == ON))	||
					((day == 7) && (alarmSunday.state == ON))) {
						// The things to do if the alarm clock is enabled for this day of week: 
						callScript("alarm2")
				   }
			}			
			]
		logInfo("rules","Alarm, timer1 set.")
		// check if there is already an alarm timer2; cancel it if present
		if (timer2 != null) {
		   timer2.cancel
		   timer2 = null
		}
		
		// create a new timer using the calculated delta [min]
		timer2 = createTimer(now.plusMinutes(delta3)) [|
			// This code will be executed if the timer triggers
			// ************************************************
			// check if alarm clock is enabled and armed for this weekday
			
			// Re-Arm the timer to trigger again tomorrow (same time) 
			timer2.reschedule(now.plusHours(24))
			
			if(alarmEnabled.state == ON)
			{
				var Number day = now.getDayOfWeek
				if (((day == 1) && (alarmMonday.state == ON))	||
					((day == 2) && (alarmTuesday.state == ON))	||
					((day == 3) && (alarmWensday.state == ON))	||
					((day == 4) && (alarmThursday.state == ON))	||
					((day == 5) && (alarmFriday.state == ON))	||
					((day == 6) && (alarmSaturday.state == ON))	||
					((day == 7) && (alarmSunday.state == ON))) {
						// The things to do if the alarm clock is enabled for this day of week: 
						callScript("alarm3")
				   }
			}			
			]
		logInfo("rules","Alarm, timer2 set.")
		// check if there is already an alarm timer3; cancel it if present
		if (timer3 != null) {
		   timer3.cancel
		   timer3 = null
		}
		
		// create a new timer using the calculated delta [min]
		timer3 = createTimer(now.plusMinutes(delta4)) [|
			// This code will be executed if the timer triggers
			// ************************************************
			// check if alarm clock is enabled and armed for this weekday
			
			// Re-Arm the timer to trigger again tomorrow (same time) 
			timer3.reschedule(now.plusHours(24))
			
			if(alarmEnabled.state == ON)
			{
				var Number day = now.getDayOfWeek
				if (((day == 1) && (alarmMonday.state == ON))	||
					((day == 2) && (alarmTuesday.state == ON))	||
					((day == 3) && (alarmWensday.state == ON))	||
					((day == 4) && (alarmThursday.state == ON))	||
					((day == 5) && (alarmFriday.state == ON))	||
					((day == 6) && (alarmSaturday.state == ON))	||
					((day == 7) && (alarmSunday.state == ON))) {
					// The things to do if the alarm clock is enabled for this day of week: 
					callScript("alarm4")
			   }
			}			
			]
		logInfo("rules","Alarm, timer3 set.")
	  } finally  {
		 // release the lock - we are ready to process the next event
		 lock1.unlock()
		 }
	 
  }//IF
end

rule "Initialization"
when 
	System started
then
	 postUpdate(alarmEnabled, ON)
	 postUpdate(radioEnabled, ON)
	 postUpdate(alarmTimeHour,  6)
	 postUpdate(alarmTimeMinutes, 30)
	 postUpdate(alarmOffset1Minutes, 20)
	 postUpdate(alarmOffset2Minutes, 15)
	 postUpdate(alarmOffset3Minutes, 50)
	 postUpdate(alarmMonday, ON)
	 postUpdate(alarmTuesday, ON)
	 postUpdate(alarmWensday, ON)
	 postUpdate(alarmThursday, ON)
	 postUpdate(alarmFriday, ON)
	 postUpdate(alarmSaturday, OFF)
	 postUpdate(alarmSunday, OFF)
end
```
# Sitemap
```
Frame label="Alarms and Timers" {

        Text label="Alarm [%s]" item=alarmTimeMessage icon="clock" {
            Frame label="Time" {
				Switch 	item=alarmEnabled
				Switch	item=radioEnabled
                Setpoint item=alarmTimeHour minValue=0 maxValue=23 step=1
                Setpoint item=alarmTimeMinutes minValue=0 maxValue=55 step=5
				Setpoint item=alarmOffset1Minutes minValue=0 maxValue=55 step=5
				Setpoint item=alarmOffset2Minutes minValue=0 maxValue=55 step=5
				Setpoint item=alarmOffset3Minutes minValue=0 maxValue=55 step=5				
            }
            Frame label="Weekdays" {
                Switch item=alarmMonday
                Switch item=alarmTuesday
                Switch item=alarmWensday
                Switch item=alarmThursday
                Switch item=alarmFriday
                Switch item=alarmSaturday
                Switch item=alarmSunday
            }
        }
    }
```

# References
 - [1]: [How to configure openHAB for setting alarm/schedule times thru the UI](https://groups.google.com/d/msg/openhab/F2hqVwjbRC0/b1s44_MWGygJ)