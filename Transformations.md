Overview of available transformation services

### Exec transformation service

The Exec transformation service will pass a value to an external program. The output of the program is used as the transformed value. A placeholder of `%s` is used to pass the value to be transformed to the external program.

Example item configuration:

`http="<[http://example.com/var/value:60000:EXEC(/absolute/path/to/my/program %s)]" }`

Example in rules:

`transform("EXEC", "/absolute/path/to/my/program %s", "foo bar baz")`

The program must either be in the executable search path of the openHAB server or an absolute path can be used.

Debugging:

Turn on DEBUG logging for package `org.openhab.io.net.exec` in `logback.xml`.

### Java Script transformation service
OpenHab supports transformation scripts written in Javascript. 

Example item configuration:

`http="<[http://example.com/var/value:60000:JS(getValue.js)]" }`

Example in rules:

`transform("JS", "getValue.js", "foo bar baz")`

Let's assume we have received a string containing `foo bar baz` and we're looking for a length of the last word (`baz`).

`getValue.js`:

    // Wrap everything in a function
    (function(i) {
        var
            array = i.split(i);
        return array[array.length - 1].length;
    })(input)
    // input variable contains data passed by openhab

### Map transformation service

Example of MAP transform inside a rule to translate weather condition:

`String wuToday "Heute: [MAP(weather_de.map):%s]" {weather="locationId=home-WU, forecast=0, type=condition, property=commonId"}`

To translate the commonID to german using the supplied MAP file use the following command in the rule:

`var String wetter = transform("MAP","weather_de.map",wuToday.state.toString)`

Important: The input variable has to be of type String!

### Scale transformation service

Example item configuration: 

`Number WuUvCode "UV Level [SCALE(uvindex.scale):%s]"`

uvindex.scale in transform folder :
```
[0,3[=1
[3,6[=2
[6,8[=3
[8,10[=4
[10,100]=5
```

Calling it in a rule :

`var Number r = transform("SCALE", "uvindex.scale", WuUvIndex.state.toString)`

### RegEx transformation service

The Regex transformation service matches a string against a regular expression with at least one capturing group. The first capturing group becomes the result of the transformation. Note that the provided regular expression will be anchored at the beginning and end of the string ("^" and "$" will be added around the provided expression).

Example item configuration:

Returns the string.<br>
`http="<[http://example.com/var/value:60000:REGEX((.*)]" }`

Extract a integer (e.g., "Result: 123" ==> "123")<br>
`http="<[http://example.com/var/value:60000:REGEX(.*?([0-9]+).*]" }`


Example in rules:

`transform("REGEX", "STATUS=(.*)", "STATUS=OK")`

### XPath transformation service
tbd ...

### XSLT transformation service
tbd ...

For samples see [here](https://github.com/openhab/openhab/wiki/Samples-XSLT-Transformations).

### JSonPath transformation service
As with XPath on XML data, [JSONPath](http://goessner.net/articles/JsonPath/) allows direct query of JSON data.

```
// Test JSONPath
rule "Test JSONPath"
when Time cron "0 * * * * ?"
then
   var String json = '{"store":{"book":[{"category":"reference","author":"Nigel Rees","title": "Sayings of the Century", "price": 8.95  } ],  "bicycle": { "color": "red",  "price": 19.95} }}' 
    var test = transform("JSONPATH","$.store.book[0].author",json)
    println(test)
end
```

As an example [Outback](http://www.outbackpower.com/) MATE3 Solar controller produces the following JSON:

```
{"devstatus": {
"Sys_Time": 1423858028,
"Sys_Batt_V": 50.5,
"ports": [
{ "Port": 1, "Dev": "GS","Type": "60Hz","Inv_I_L1": 1,"Chg_I_L1": 0,"Buy_I_L1": 0,"Sell_I_L1": 0,"VAC1_in_L1": 15,"VAC2_in_L1": 0,"VAC_out_L1": 113,"Inv_I_L2": 2,"Chg_I_L2": 0,"Buy_I_L2": 0,"Sell_I_L2": 0,"VAC1_in_L2": 3,"VAC2_in_L2": 0,"VAC_out_L2": 114,"AC_Input": "Gen","Batt_V": 50.0,"AC_mode": "NO AC","INV_mode": "Inverting","Warn": ["none"],"Error": ["none"],"AUX": "disabled","RELAY": "disabled"},
{ "Port": 2, "Dev": "GS","Type": "60Hz","Inv_I_L1": 0,"Chg_I_L1": 0,"Buy_I_L1": 0,"Sell_I_L1": 0,"VAC1_in_L1": 16,"VAC2_in_L1": 0,"VAC_out_L1": 114,"Inv_I_L2": 0,"Chg_I_L2": 0,"Buy_I_L2": 0,"Sell_I_L2": 0,"VAC1_in_L2": 3,"VAC2_in_L2": 0,"VAC_out_L2": 113,"AC_Input": "Gen","Batt_V": 49.6,"AC_mode": "NO AC","INV_mode": "Inverting","Warn": ["none"],"Error": ["none"],"AUX": "disabled","RELAY": "disabled"},
{ "Port": 4, "Dev": "CC","Type": "FM","Out_I": 0.0,"In_I": 0,"Batt_V": 50.6,"In_V": 25.1,"Out_kWh": 15.5,"Out_AH": 281,"CC_mode": "Silent","Error": ["none"],"Aux_mode": "Manual","AUX": "disabled"},
{ "Port": 5, "Dev": "CC","Type": "FM","Out_I": 0.0,"In_I": 0,"Batt_V": 50.6,"In_V": 22.2,"Out_kWh": 23.6,"Out_AH": 433,"CC_mode": "Silent","Error": ["none"],"Aux_mode": "Manual","AUX": "disabled"},
{ "Port": 6, "Dev": "FNDC","Enabled": ["A","B"],"Shunt_A_I":  -10.3,"Shunt_A_AH": -62,"Shunt_A_kWh":  -3.120,"Shunt_B_I": -0.1,"Shunt_B_AH": 11,"Shunt_B_kWh":  0.580,"SOC": 96,"Min_SOC": 70,"Days_since_full": 42.1,"CHG_parms_met": false,"In_AH_today": 731,"Out_AH_today": 608,"In_kWh_today":  39.520,"Out_kWh_today":  30.890,"Net_CFC_AH": -52,"Net_CFC_kWh":  -2.570,"Batt_V": 50.5,"Batt_temp": "14 C","Aux_mode": "auto","AUX": "disabled"}
]}}
```

To pull the battery State of Charge (SOC) using JsonPath:

```
Number	Battery_Charge	"Battery Charge"	(Solar)	{http="<[http://{ip}/Dev_status.cgi&Port=0:30000:JSONPATH($.devstatus.ports[?(@.Port==6)][0].SOC)]"}
```
