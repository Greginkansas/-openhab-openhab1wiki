The example can be used to convert a Fahrenheit temperature value to Celsius:
```Xtend
import org.openhab.core.library.types.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*

rule "Convert_Temp_EG"
when
	Item ZwaveTemperatureEGF changed 
then
    var tempFahrenheit = (ZwaveTemperatureEGF.state as DecimalType).doubleValue
	var tempCelsius = (tempFahrenheit -  32)  *  5/9
	postUpdate(ZwaveTemperatureEG, tempCelsius)
end
```

This example can be used to convert Celsius to Fahrenheit, mostly useful for Fibraro motion sensors : 
```Xtend
import org.openhab.core.library.types.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*
rule "Convert ZwaveTempC to F"
	when
		Item ZwaveTempC changed
	then
		var Number temp
		if (ZwaveTempC.state instanceof DecimalType) temp = ZwaveTempC.state as DecimalType
		var Number temp2 = (temp * 1.8) + 32
		ZwaveTempF.postUpdate(temp2)
	end
```