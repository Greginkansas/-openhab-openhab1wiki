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

*  [Example IV] (AlarmClock#7-day-three-zone-scheduler-with-boost-buttons---example-iv)
 * [Items] (AlarmClock#items-3)
 * [Rules] (AlarmClock#rules-3)
 * [Sitemap] (AlarmClock#sitemap-3)

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
# 7 Day Three Zone Scheduler with Boost Buttons - Example IV
I based this on the previous examples. I'm working on a smart thermostat for my house, but openhab doesn't supply a 7 day 3 zone widget out of the box. The code probably needs a tidy up and there is possibly a better way to approach the issue but this solves a lot of problems. Mainly, due to the number of timers involved I needed to use java 8 lambdas to supply a method for scheduling, boosting and managing the relays which could be reused in order to keep the size of the rules file down. One bug i've noticed is that it doesn't deal well with DST on the first day of the time switch, but then resolves to the correct time on the subsequent days. 

## Items 
```
Group gTimerButtons
Group gTimerButtonsTimeout
Group gWeekDays
Group gDownWkSchedTime
Group gUpWkSchedTime
Group gWaterWkSchedTime



Switch ADownstairs "Main Downstairs Switch [%s]" <groundfloor> { gpio="pin:16 activelow:yes force:yes"}
Switch AUpstairs "Main Upstairs Switch [%s]" <firstfloor> { gpio="pin:20 activelow:yes force:yes"}
Switch AWater "Main Hot Water Switch [%s]" <cistern> { gpio="pin:21 activelow:yes force:yes"}

Switch DownBoostSwitch "Boost Ground Floor" <groundfloor> (gTimerButtons)
Switch UpstairsBoostSwitch "Boost Upstairs" <firstfloor> (gTimerButtons) 
Switch WaterBoostSwitch "Boost Hot Water" <cistern> (gTimerButtons) 

Number DownstairsBoostTime "Hours [%d]" <clock-on> (gTimerButtons)
Number UpstairsBoostTime "Hours [%d]" <clock-on> (gTimerButtons)
Number WaterBoostTime "Hours [%d]" <clock-on> (gTimerButtons)



Switch DownSchedSwitch
Switch UpstairsSchedSwitch
Switch WaterSchedSwitch

Switch EnableDownstairsScheduler
Switch EnableUpstairsScheduler
Switch EnableWaterScheduler


Switch  DownWkSchedMonday "Monday"     <switch>  (gDownWkSchedTime) 
Switch  DownWkSchedTuesday "Tuesday"   <switch>  (gDownWkSchedTime) 
Switch  DownWkSchedWednesday "Wednesday"   <switch>  (gDownWkSchedTime) 
Switch  DownWkSchedThursday "Thursday" <switch>  (gDownWkSchedTime) 
Switch  DownWkSchedFriday "Friday"    <switch>  (gDownWkSchedTime) 
Switch  DownWkSchedSaturday "Saturday"    <switch>  (gDownWkSchedTime) 
Switch  DownWkSchedSunday "Sunday"    <switch>  (gDownWkSchedTime) 

String DownWkSchedTimeStartMessage "%s"
String DownWkSchedTimeEndMessage  "%s"
Number DownWkSchedTimeHour "Hour [%d]" <clock-on> (gDownWkSchedTime)
Number DownWkSchedTimeMinute "Minute [%d]" <clock-on> (gDownWkSchedTime)
Number DownWkSchedTimeEndHour "Hour [%d]" <clock> (gDownWkSchedTime)
Number DownWkSchedTimeEndMinute "Minute [%d]" <clock> (gDownWkSchedTime)


Switch  UpWkSchedMonday "Monday"     <switch>  (gUpWkSchedTime) 
Switch  UpWkSchedTuesday "Tuesday"   <switch>  (gUpWkSchedTime) 
Switch  UpWkSchedWednesday "Wednesday"   <switch>  (gUpWkSchedTime) 
Switch  UpWkSchedThursday "Thursday" <switch>  (gUpWkSchedTime) 
Switch  UpWkSchedFriday "Friday"    <switch>  (gUpWkSchedTime) 
Switch  UpWkSchedSaturday "Saturday"    <switch>  (gUpWkSchedTime) 
Switch  UpWkSchedSunday "Sunday"    <switch>  (gUpWkSchedTime) 

String UpWkSchedTimeStartMessage "%s"
String UpWkSchedTimeEndMessage  "%s"
Number UpWkSchedTimeHour "Hour [%d]" <clock-on> (gUpWkSchedTime)
Number UpWkSchedTimeMinute "Minute [%d]" <clock-on> (gUpWkSchedTime)
Number UpWkSchedTimeEndHour "Hour [%d]" <clock> (gUpWkSchedTime)
Number UpWkSchedTimeEndMinute "Minute [%d]" <clock> (gUpWkSchedTime)


Switch  WaterWkSchedMonday "Monday"     <switch>  (gWaterWkSchedTime) 
Switch  WaterWkSchedTuesday "Tuesday"   <switch>  (gWaterWkSchedTime) 
Switch  WaterWkSchedWednesday "Wednesday"   <switch>  (gWaterWkSchedTime) 
Switch  WaterWkSchedThursday "Thursday" <switch>  (gWaterWkSchedTime) 
Switch  WaterWkSchedFriday "Friday"    <switch>  (gWaterWkSchedTime) 
Switch  WaterWkSchedSaturday "Saturday"    <switch>  (gWaterWkSchedTime) 
Switch  WaterWkSchedSunday "Sunday"    <switch>  (gWaterWkSchedTime) 

String WaterWkSchedTimeStartMessage "%s"
String WaterWkSchedTimeEndMessage  "%s"
Number WaterWkSchedTimeHour "Hour [%d]" <clock-on> (gWaterWkSchedTime)
Number WaterWkSchedTimeMinute "Minute [%d]" <clock-on> (gWaterWkSchedTime)
Number WaterWkSchedTimeEndHour "Hour [%d]" <clock> (gWaterWkSchedTime)
Number WaterWkSchedTimeEndMinute "Minute [%d]" <clock> (gWaterWkSchedTime)
```

## Sitemap 
```
sitemap default label="The Qstat"
{
	
	Frame label="Current Status"
	{
		Text item=ADownstairs 
		Text item=AUpstairs 
		Text item=AWater 

	}

	Frame label="Booster Buttons"
	{
	            	
	    Selection item=DownstairsBoostTime label="Downstairs Boost Time" mappings=[30="0.5 Hours", 60="1 Hour", 90="1.5 Hours", 120="2 Hours", 150="2.5 Hours", 180="3 Hours", 210="3.5 hours", 240="4 Hours", 300="5 Hours"]
		Switch item=DownBoostSwitch
		Selection item=UpstairsBoostTime label="Upstairs Boost Time" mappings=[30="0.5 Hours", 60="1 Hour", 90="1.5 Hours", 120="2 Hours", 150="2.5 Hours", 180="3 Hours", 210="3.5 hours", 240="4 Hours", 300="5 Hours"]
		Switch item=UpstairsBoostSwitch
		Selection item=WaterBoostTime label="Water Boost Time" mappings=[30="0.5 Hours", 60="1 Hour"]
		Switch item=WaterBoostSwitch
	}
	
	
    Frame label="Heating Schedules" {
    	Text label="Downstairs" icon="clock-on" {
    	
	            Frame label="Enable Downstairs Schedule" {
	                Switch item=EnableDownstairsScheduler label="Enable Scheduling"
	            	
	            }
	            Frame label="Start Time" {
	            	Selection item=DownWkSchedTimeHour label="Select Hour" mappings=[0="00:00", 1="01:00", 2="02:00", 3="03:00", 4="04:00", 5="05:00", 6="06:00", 7="07:00", 8="08:00", 9="09:00", 10="10:00", 11="11:00", 12="12:00",13="13:00", 14="14:00", 15="15:00", 16="16:00", 17="17:00", 18="18:00", 19="19:00", 20="20:00", 21="21:00", 22="22:00", 23="23:00"]
	            	Selection item=DownWkSchedTimeMinute label="Select Minutes" mappings=[0="00", 5="05", 10="10", 15="15", 20="20", 25="25", 30="30", 35="35", 40="40", 45="45", 50="50", 55="55"]
	            	
	            }
	            Frame label="End Time" {
	            	Selection item=DownWkSchedTimeEndHour label="Select Hour" mappings=[0="00:00", 1="01:00", 2="02:00", 3="03:00", 4="04:00", 5="05:00", 6="06:00", 7="07:00", 8="08:00", 9="09:00", 10="10:00", 11="11:00", 12="12:00",13="13:00", 14="14:00", 15="15:00", 16="16:00", 17="17:00", 18="18:00", 19="19:00", 20="20:00", 21="21:00", 22="22:00", 23="23:00"]
	            	Selection item=DownWkSchedTimeEndMinute label="Select Minutes" mappings=[0="00", 5="05", 10="10", 15="15", 20="20", 25="25", 30="30", 35="35", 40="40", 45="45", 50="50", 55="55"]
	            	
	            }	    
	            Frame label="Days Enabled" {
	                Switch item=DownWkSchedMonday
	                Switch item=DownWkSchedTuesday
	                Switch item=DownWkSchedWednesday
	                Switch item=DownWkSchedThursday
	                Switch item=DownWkSchedFriday
	                Switch item=DownWkSchedSaturday
	                Switch item=DownWkSchedSunday
	            }
        }
    	Text label="Upstairs" icon="clock-on" {
    	
	            Frame label="Enable Upstairs Schedule" {
	                Switch item=EnableUpstairsScheduler label="Enable Scheduling"
	            	
	            }
	            Frame label="Start Time" {
	            	Selection item=UpWkSchedTimeHour label="Select Hour" mappings=[0="00:00", 1="01:00", 2="02:00", 3="03:00", 4="04:00", 5="05:00", 6="06:00", 7="07:00", 8="08:00", 9="09:00", 10="10:00", 11="11:00", 12="12:00",13="13:00", 14="14:00", 15="15:00", 16="16:00", 17="17:00", 18="18:00", 19="19:00", 20="20:00", 21="21:00", 22="22:00", 23="23:00"]
	            	Selection item=UpWkSchedTimeMinute label="Select Minutes" mappings=[0="00", 5="05", 10="10", 15="15", 20="20", 25="25", 30="30", 35="35", 40="40", 45="45", 50="50", 55="55"]
	            	
	            }
	            Frame label="End Time" {
	            	Selection item=UpWkSchedTimeEndHour label="Select Hour" mappings=[0="00:00", 1="01:00", 2="02:00", 3="03:00", 4="04:00", 5="05:00", 6="06:00", 7="07:00", 8="08:00", 9="09:00", 10="10:00", 11="11:00", 12="12:00",13="13:00", 14="14:00", 15="15:00", 16="16:00", 17="17:00", 18="18:00", 19="19:00", 20="20:00", 21="21:00", 22="22:00", 23="23:00"]
	            	Selection item=UpWkSchedTimeEndMinute label="Select Minutes" mappings=[0="00", 5="05", 10="10", 15="15", 20="20", 25="25", 30="30", 35="35", 40="40", 45="45", 50="50", 55="55"]
	            	
	            }	    
	            Frame label="Days Enabled" {
	                Switch item=UpWkSchedMonday
	                Switch item=UpWkSchedTuesday
	                Switch item=UpWkSchedWednesday
	                Switch item=UpWkSchedThursday
	                Switch item=UpWkSchedFriday
	                Switch item=UpWkSchedSaturday
	                Switch item=UpWkSchedSunday
	            }
	        
        }
    	Text label="Hot Water" icon="clock-on" {
 
 	            Frame label="Enable Water Schedule" {
	                Switch item=EnableWaterScheduler label="Enable Scheduling"
	            	
	            }   	

	            Frame label="Start Time" {
	            	Selection item=WaterWkSchedTimeHour label="Select Hour" mappings=[0="00:00", 1="01:00", 2="02:00", 3="03:00", 4="04:00", 5="05:00", 6="06:00", 7="07:00", 8="08:00", 9="09:00", 10="10:00", 11="11:00", 12="12:00",13="13:00", 14="14:00", 15="15:00", 16="16:00", 17="17:00", 18="18:00", 19="19:00", 20="20:00", 21="21:00", 22="22:00", 23="23:00"]
	            	Selection item=WaterWkSchedTimeMinute label="Select Minutes" mappings=[0="00", 5="05", 10="10", 15="15", 20="20", 25="25", 30="30", 35="35", 40="40", 45="45", 50="50", 55="55"]
	            	
	            }
	            Frame label="End Time" {
	            	Selection item=WaterWkSchedTimeEndHour label="Select Hour" mappings=[0="00:00", 1="01:00", 2="02:00", 3="03:00", 4="04:00", 5="05:00", 6="06:00", 7="07:00", 8="08:00", 9="09:00", 10="10:00", 11="11:00", 12="12:00",13="13:00", 14="14:00", 15="15:00", 16="16:00", 17="17:00", 18="18:00", 19="19:00", 20="20:00", 21="21:00", 22="22:00", 23="23:00"]
	            	Selection item=WaterWkSchedTimeEndMinute label="Select Minutes" mappings=[0="00", 5="05", 10="10", 15="15", 20="20", 25="25", 30="30", 35="35", 40="40", 45="45", 50="50", 55="55"]
	            	
	            }	    
	            Frame label="Days Enabled" {
	                Switch item=WaterWkSchedMonday
	                Switch item=WaterWkSchedTuesday
	                Switch item=WaterWkSchedWednesday
	                Switch item=WaterWkSchedThursday
	                Switch item=WaterWkSchedFriday
	                Switch item=WaterWkSchedSaturday
	                Switch item=WaterWkSchedSunday
	            }
	        }  
	}
	
	Frame label="Expert settings"
	{

		Text label="Overrides" icon="settings"	{
			
			Switch item=ADownstairs label="Hard Downstairs Switch"
			Switch item=AUpstairs label="Hard Upstairs Switch"
			Switch item=AWater label="Hard Water Switch"
		}
	}


}


```
## Rules 
```
import java.util.List
import java.util.Map
import java.util.concurrent.locks.ReentrantLock
import org.openhab.action.squeezebox.*
import org.openhab.core.library.types.*
import org.openhab.core.library.items.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*
import org.openhab.model.script.actions.*
import org.openhab.io.myopenhab.MyOpenHABAction
import org.joda.time.*


var java.util.concurrent.locks.ReentrantLock lock1 = new java.util.concurrent.locks.ReentrantLock()
val Map<String, NumberItem> dAlarmTimes = newHashMap
val Map<String, Timer> dStartTimers = newHashMap
val List<SwitchItem> dDaySwitches = newArrayList

val Map<String, NumberItem> uAlarmTimes = newHashMap
val Map<String, Timer> uStartTimers = newHashMap
val List<SwitchItem> uDaySwitches = newArrayList

val Map<String, NumberItem> wAlarmTimes = newHashMap
val Map<String, Timer> wStartTimers = newHashMap
val List<SwitchItem> wDaySwitches = newArrayList

val Map<String, Timer> boostTimers = newHashMap

val String START_TIME_HOUR = "StartHour"
val String END_TIME_HOUR = "EndHour"
val String START_TIME_MINUTE = "StartMinute" 
val String END_TIME_MINUTE = "EndMinute"

val org.eclipse.xtext.xbase.lib.Functions$Function4 boostMethod = [SwitchItem boostSwitch,
																		Map <String, Timer>  boostMaps,						
																		String boostName,
																		NumberItem timeToBoost|
		
		try {
			var Timer boostTimer = boostMaps?.get(boostName)
		
	  		if(boostSwitch.state == ON && (boostTimer == null || boostTimer.hasTerminated )) {
	  			
	    
		    	var DateTime boostTime = now.plusMinutes((timeToBoost.state as DecimalType).intValue)
		    	var String tempDisplayString = String::format("%02d:%02d", boostTime.getHourOfDay(), boostTime.getMinuteOfHour())
			   	logInfo(boostName + " Boost: ","***** Stop at " + tempDisplayString + " *****")
			   	timeToBoost.postUpdate(60)
			   	
			   	sendBroadcastNotification(boostName + " is now on until " + tempDisplayString + "!!")
		
		        boostTimer = createTimer(boostTime, [|
		        	
			   		 logInfo(boostName + " Boost: ","***** Finished *****")
		        	
			   		 boostSwitch.sendCommand(OFF)
		        ])
		        
			    boostMaps.put(boostName, boostTimer)
	        
	        }
	    	
	    	
	    	if (boostSwitch.state == OFF) {
	    	
			   	logInfo(boostName + " Boost: ","***** Cancelling boost *****")
		        boostTimer?.cancel
		        boostTimer = null
		        boostMaps.put(boostName, boostTimer)
	        
	    	}
	  catch (Throwable t) {
	  			boostSwitch.sendCommand(OFF)
	  	
	  }
]


val org.eclipse.xtext.xbase.lib.Functions$Function5 manageRelays = [SwitchItem hardSwitch,
																	SwitchItem boostSwitch,
																	SwitchItem schedulerSwitch,
																	SwitchItem floorTimerSwitch,
																		String floorName|
	
    	
    	
   	if(boostSwitch.state == ON || (floorTimerSwitch.state == ON && schedulerSwitch.state == ON)) {
    
		hardSwitch.sendCommand(ON)
		//sendBroadcastNotification(floorName + "  is now on!!")
		
    }
    else {
    	
    	if (hardSwitch.state == ON) {
    		sendBroadcastNotification(floorName + "  is now off!!")
    		
    	}
		hardSwitch.sendCommand(OFF)
	
   }
]


  	
  	
val org.eclipse.xtext.xbase.lib.Functions$Function5 scheduleHeating = [SwitchItem relayItem,
																		Map<String,Timer> timerMap,
																		Map<String, NumberItem> alarmTimes,
																		List<SwitchItem> daySwitches,
																		String name|
    
  	try {
  		
  		
		var selStartHour =  alarmTimes.get("StartHour")		
	    var decStartHour = selStartHour.state as DecimalType    
	    var selStartMinute = alarmTimes.get("StartMinute")
	    var decStartMinute = selStartMinute.state as DecimalType
	
	    // calculate the alarm time [min]
	    var int wakeTime1
	    wakeTime1 = (selStartHour.state as DecimalType).intValue * 3600 + 
	                60 * (selStartMinute.state as DecimalType).intValue
	    wakeTime1 = wakeTime1.intValue
	
	    // calculate current time [min]
	    var int startTimeInMins
	    startTimeInMins = now.getSecondOfDay
	    startTimeInMins = startTimeInMins.intValue
	
	    // calculate the difference between the requested alarm time and 
	    // current time (again in minutes)  
	    var int startDelta
	    startDelta = (wakeTime1 - startTimeInMins)
	    startDelta = startDelta.intValue
	
	    // add one day (1440 minutes) if alarm time for today already passed
	    if (startTimeInMins > wakeTime1) { startDelta = startDelta + (1440 * 60) }
	
	    // check if there is already an alarm timer; cancel it if present
		timerMap.get("StartTimerKey")?.cancel
	    timerMap.put("StartTimerKey", null) 	

	    
	    // create a new timer using the calculated delta [min]
	    logInfo("Scheduler: ","***** " + name + " Starting *****")
	    
	    timerMap.put("StartTimerKey", createTimer(now.plusSeconds(startDelta)) [|
	        // This code will be executed if the timer triggers
	        // ************************************************
	        // check if alarm clock is armed for this weekday

	        
	        var Number day = now.getDayOfWeek	       
	        
	        if (((day == 1) && (daySwitches.get(0).state == ON))   ||
	            ((day == 2) && (daySwitches.get(1).state == ON))   ||
	            ((day == 3) && (daySwitches.get(2).state == ON))   ||
	            ((day == 4) && (daySwitches.get(3).state == ON))   ||
	            ((day == 5) && (daySwitches.get(4).state == ON))   ||
	            ((day == 6) && (daySwitches.get(5).state == ON))   ||
	            ((day == 7) && (daySwitches.get(6).state == ON))) {
	            	
	            	
	            // Dealing with end time 
		   		var String endMsg = ""
			
				var endHour = alarmTimes.get("EndHour")
			    var dEndHour = endHour.state as DecimalType
			    
			    var endMinute = alarmTimes.get("EndMinute")
			    var dEndMinute = endMinute.state as DecimalType
			
			    // Combine the hour and minutes to one string to be displayed in the 
			    // user interface
			    if (dEndHour < 10) { endMsg = "0" } 
			    endMsg = endMsg + endHour.state.format("%d") + ":"
			
			    if (dEndMinute < 10) { endMsg = endMsg + "0" }
			    endMsg = endMsg + endMinute.state.format("%d")
			
			    // calculate the alarm time [min]
			    var int endTime1
			    endTime1 = (endHour.state as DecimalType).intValue * 3600 + 
			                60 * (endMinute.state as DecimalType).intValue
			    endTime1 = endTime1.intValue
			
			    // calculate current time [min]
			    var int endTimeInMins
			    endTimeInMins = now.getSecondOfDay
			    endTimeInMins = endTimeInMins.intValue
			
			    // calculate the difference between the requested alarm time and 
			    // current time (again in minutes)  
			    var int endDelta
			    endDelta = (endTime1 - endTimeInMins)
			    endDelta = endDelta.intValue
			
			    // add one day (1440 minutes) if alarm time for today already passed
			    if (endTimeInMins > endTime1) { endDelta = endDelta + (1440 * 60) }
	
	   			logInfo("Scheduler: ","***** " + name + " on until " + endMsg + "*****"
			    
				sendBroadcastNotification(name + " is now on until " + endMsg + "!!")
				
				relayItem.sendCommand(ON)
				timerMap.put("EndTimerKey", createTimer(now.plusSeconds(endDelta)) [|
					
					relayItem.sendCommand(OFF)
					timerMap.get("EndTimerKey")?.cancel
	    			timerMap.put("EndTimerKey", null)
					
				])
				
	           }
	            // Re-Arm the timer to trigger again tomorrow (same time) 
	           timerMap.get("StartTimerKey").reschedule(now.plusHours(24))

	           
	           

	        // ************************************************
	        // Here the code ends that executes once the timer triggers 
	        ])
	  } 
	  catch (Throwable t) {
	  			hardSwitch.sendCommand(OFF)
	  	
	  	}
	  	
	  finally  {
     // release the lock - we are ready to process the next event
  }
]




rule "Downstairs Booster"
when
    Item DownBoostSwitch received command ON or
    Item DownBoostSwitch received command OFF 
    
    
then

    lock1.lock()
    boostMethod.apply(DownBoostSwitch, boostTimers, "Downstairs", DownstairsBoostTime)
    lock1.unlock()
end


rule "Upstairs Booster"
when
    Item UpstairsBoostSwitch received command ON or
    Item UpstairsBoostSwitch received command OFF
    
then
    lock1.lock()
    boostMethod.apply(UpstairsBoostSwitch, boostTimers, "Upstairs", UpstairsBoostTime)
    lock1.unlock()
end


rule "Hotwater Booster"
when
    Item WaterBoostSwitch received command ON or
    Item WaterBoostSwitch received command OFF
then
    lock1.lock()
    boostMethod.apply(WaterBoostSwitch, boostTimers, "Water", WaterBoostTime)
    lock1.unlock()
end

rule "Actual Downstairs"
when
    Item DownBoostSwitch received update  or
    Item DownSchedSwitch received update or
    Item EnableDownstairsScheduler received update


then
    // save current states into vals
	lock1.lock()
    manageRelays.apply(ADownstairs,DownBoostSwitch,EnableDownstairsScheduler, DownSchedSwitch, "Downstairs Heating")
	lock1.unlock()
end


rule "Actual Upstairs"
when
    Item UpstairsBoostSwitch received update  or
    Item UpstairsSchedSwitch received update or 
    Item EnableUpstairsScheduler received update
then
	lock1.lock()
    manageRelays.apply(AUpstairs,UpstairsBoostSwitch,EnableUpstairsScheduler, UpstairsSchedSwitch, "Upstairs Heating")
	lock1.unlock()
end

rule "Actual Water"
when
    Item WaterBoostSwitch received update  or
    Item WaterSchedSwitch received update or
    Item EnableWaterScheduler received update
    
then
    lock1.lock()
    manageRelays.apply(AWater,WaterBoostSwitch,EnableWaterScheduler, WaterSchedSwitch, "Hot Water")
	lock1.unlock()
end





rule "Initialization"
 when 
   System started
 then
         lock1.lock()
     
     postUpdate(DownWkSchedTimeHour,  17)
     postUpdate(DownWkSchedTimeMinute, 30)
     postUpdate(DownWkSchedTimeEndHour,  19)
     postUpdate(DownWkSchedTimeEndMinute, 00)
     
     postUpdate(DownWkSchedMonday,     OFF)
     postUpdate(DownWkSchedTuesday,   OFF)
     postUpdate(DownWkSchedWednesday,   OFF)
     postUpdate(DownWkSchedThursday, OFF)
     postUpdate(DownWkSchedFriday,    OFF)
     postUpdate(DownWkSchedSaturday,    OFF)
     postUpdate(DownWkSchedSunday,    OFF)
     
     
     dDaySwitches.add(DownWkSchedMonday);
 	 dDaySwitches.add(DownWkSchedTuesday);
 	 dDaySwitches.add(DownWkSchedWednesday);
 	 dDaySwitches.add(DownWkSchedThursday);
 	 dDaySwitches.add(DownWkSchedFriday);
 	 dDaySwitches.add(DownWkSchedSaturday);
 	 dDaySwitches.add(DownWkSchedSunday);
 	 
	 dAlarmTimes.put(START_TIME_HOUR, DownWkSchedTimeHour)
	 dAlarmTimes.put(START_TIME_MINUTE, DownWkSchedTimeMinute)	
 	 dAlarmTimes.put(END_TIME_HOUR, DownWkSchedTimeEndHour)	
 	 dAlarmTimes.put(END_TIME_MINUTE, DownWkSchedTimeEndMinute)	
 
	 
	 //////////////////////////////////////
	 
	 postUpdate(UpWkSchedTimeHour,  7)
     postUpdate(UpWkSchedTimeMinute, 00)
     postUpdate(UpWkSchedTimeEndHour,  8)
     postUpdate(UpWkSchedTimeEndMinute, 00)
     
     postUpdate(UpWkSchedMonday,     OFF)
     postUpdate(UpWkSchedTuesday,   OFF)
     postUpdate(UpWkSchedWednesday,   OFF)
     postUpdate(UpWkSchedThursday, OFF)
     postUpdate(UpWkSchedFriday,    OFF)
     postUpdate(UpWkSchedSaturday,    OFF)
     postUpdate(UpWkSchedSunday,    OFF)
     
     
     uDaySwitches.add(UpWkSchedMonday);
 	 uDaySwitches.add(UpWkSchedTuesday);
 	 uDaySwitches.add(UpWkSchedWednesday);
 	 uDaySwitches.add(UpWkSchedThursday);
 	 uDaySwitches.add(UpWkSchedFriday);
 	 uDaySwitches.add(UpWkSchedSaturday);
 	 uDaySwitches.add(UpWkSchedSunday);
 	 
	 uAlarmTimes.put(START_TIME_HOUR, UpWkSchedTimeHour)
	 uAlarmTimes.put(START_TIME_MINUTE, UpWkSchedTimeMinute)	
 	 uAlarmTimes.put(END_TIME_HOUR, UpWkSchedTimeEndHour)	
 	 uAlarmTimes.put(END_TIME_MINUTE, UpWkSchedTimeEndMinute)	

	 
	 /////////////////////////////////////
	 

     postUpdate(WaterWkSchedTimeHour,  21)
     postUpdate(WaterWkSchedTimeMinute, 00)
     postUpdate(WaterWkSchedTimeEndHour,  21)
     postUpdate(WaterWkSchedTimeEndMinute, 30)
     
     postUpdate(WaterWkSchedMonday,     OFF)
     postUpdate(WaterWkSchedTuesday,   ON)
     postUpdate(WaterWkSchedWednesday,   OFF)
     postUpdate(WaterWkSchedThursday, ON)
     postUpdate(WaterWkSchedFriday,    OFF)
     postUpdate(WaterWkSchedSaturday,    ON)
     postUpdate(WaterWkSchedSunday,    OFF)
 
 
          
     wDaySwitches.add(WaterWkSchedMonday);
 	 wDaySwitches.add(WaterWkSchedTuesday);
 	 wDaySwitches.add(WaterWkSchedWednesday);
 	 wDaySwitches.add(WaterWkSchedThursday);
 	 wDaySwitches.add(WaterWkSchedFriday);
 	 wDaySwitches.add(WaterWkSchedSaturday);
 	 wDaySwitches.add(WaterWkSchedSunday);
 	 
	 wAlarmTimes.put(START_TIME_HOUR, WaterWkSchedTimeHour)
	 wAlarmTimes.put(START_TIME_MINUTE, WaterWkSchedTimeMinute)	
 	 wAlarmTimes.put(END_TIME_HOUR, WaterWkSchedTimeEndHour)	
 	 wAlarmTimes.put(END_TIME_MINUTE, WaterWkSchedTimeEndMinute)	
 
	 
	 // Set Schedules

     
     postUpdate(EnableDownstairsScheduler,    OFF)
     postUpdate(EnableUpstairsScheduler,    OFF)
     postUpdate(EnableWaterScheduler,    ON)   

     postUpdate(DownBoostSwitch,    OFF)
     postUpdate(UpstairsBoostSwitch,    OFF)
     postUpdate(WaterBoostSwitch,    OFF)
 
     postUpdate(ADownstairs,    OFF)
     postUpdate(AUpstairs,    OFF)
     postUpdate(AWater,    OFF)
     
     postUpdate(DownstairsBoostTime,    60)
     postUpdate(UpstairsBoostTime,    60)
     postUpdate(WaterBoostTime,    60)

     
     
     
     
	 lock1.unlock()
	     
	 
	 

 end

rule "Scheduling Downstairs"
when
    Item gDownWkSchedTime received update  
then
 
    lock1.lock()
    scheduleHeating.apply(DownSchedSwitch, dStartTimers, dAlarmTimes, dDaySwitches, "Downstairs")
	lock1.unlock()
 
end

rule "Scheduling Upstairs"
when
    Item gUpWkSchedTime received update  
then
 
    lock1.lock()
    scheduleHeating.apply(UpstairsSchedSwitch, uStartTimers, uAlarmTimes, uDaySwitches, "Upstairs")
	lock1.unlock()
 
end


rule "Scheduling Water"
when
    Item gWaterWkSchedTime received update  
then
 
    lock1.lock()
    scheduleHeating.apply(WaterSchedSwitch, wStartTimers, wAlarmTimes, wDaySwitches, "Water")
	lock1.unlock()
 
end

```

## Screenshot 
![UI (greenT)](http://i.imgur.com/qlkaNI0.png)

# References
 - [1]: [How to configure openHAB for setting alarm/schedule times thru the UI](https://groups.google.com/d/msg/openhab/F2hqVwjbRC0/b1s44_MWGygJ)