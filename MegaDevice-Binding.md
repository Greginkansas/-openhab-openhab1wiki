**MegaDevice Binding**

- Working as Switch, Dimmer, Number, String 

In MegaDevice settings server string should be for example **192.168.0.1:8989** 
and any scriptname

In ports settings you should enable P+R mode.

**To change port number use Openhub config file:**

    ################################ Megadevice Binding #######################################
    #
    # IP address of a Http port for megadevice defaut value is 8989
    megadevice:httpserverport=8585


**The general principle:**

    {megadevice= mega password  : ip megaDevice : port number of megaDevice(for mega-328 is 0-13) : refresh interval"}


**EXAMPLES**

**item:**

    Dimmer DimmedLight	"Dimmer [%d %%]"	{megadevice="sec:192.168.0.14:10"}

**sitemap:**

    Slider item=DimmedLight



**For adc values use Number**

**item:**

    Number ADCPort15 "ADC value port 15 of megaDevice 1: [%d]" {megadevice="sec:192.168.0.14:15"}

**For temperature value use Number:**

	{megadevice="sec:192.168.0.17:0,dht11,t:30"}

where:

##### t- temperature
##### h- humidity. (not available for 1w)
sensors type:
##### dht11
##### dht22
##### 1w

If you have problems with 00.0 instead temperature or humidity, you can use "r" parameter. In this case values from sensors comes unparsed(raw). You can parse it youself with rules. 

**item:**

	String MegaTempHumParse         "Parse string [%s]"        {megadevice="sec:192.168.0.17:3,dht,r:30"}

**rule:**

	rule "Mega DHT Temp/Hum Parser"
	when 
	    Item MegaTempHumParse received update
	then
	    val parse = MegaTempHumParse.state.toString.split("/")
	    val parsedtemperature = parse.get(0)
	    val parsedhumidity = parse.get(1)
	    var temperature = new Double(parsedtemperature)
	    var humidity = new Double(parsedhumidity)
   
	    postUpdate(TempDHT, temperature)
	    postUpdate(HumDHT, humidity)
   
	    if(temperature == 5.0){
           logInfo("Test", "5.0")
	    }
	end

