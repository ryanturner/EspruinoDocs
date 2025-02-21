<!--- Copyright (c) 2016 Gordon Williams, Pur3 Ltd. See the file LICENSE for copying permission. -->
Logging to Google Sheets
===============================

* KEYWORDS: Wifi,Thermometer,Google Sheets,Google Forms,IoT,ESP8266,HTTPS,TLS
* USES: Internet,Pico,ESP8266,HTTPS

Introduction
-------------

Often you'll want to take some measurements and then upload them to an online service. There are [some examples of common services here](/IoT+Services), but perhaps one of the most approachable ways to get at your data is just to put it on a [Google Spreadsheet](https://www.google.com/sheets/about/), which you can do by uploading data using [Google Forms](https://www.google.com/forms/about/)

With the Espruino Pico's HTTPS support you can now do this. In this example we'll use an [[ESP8266]] WiFi module to upload the STM32 chip's temperature to Google every minute.

You'll Need
----------

* One [Espruino Pico](/Pico)
* A [[ESP8266]] WiFi module

Wiring Up
--------

Just follow [the instructions](/ESP8266) for wiring up the ESP8266 module - it's easier to use an adaptor shim for it

Software
--------

* Make sure your [[Pico]]'s firmware is up to date. You need at least `1v85` (if this hasn't been released yet, use one of the [cutting-edge builds](http://www.espruino.com/binaries/git/commits/master/)).
* Copy and paste the code below into the right-hand side of the Web IDE - changing the WiFi name and password in the `onInit` call.

```
// The 128kB of flash memory on Pico that's not supposed to exist :)
var addr = process.memory().flash_start + process.memory().flash_length;
// flash memory module
var flash = require("Flash");
// Erase all data in that flash page
flash.erasePage(addr);

/* This writes data to flash, and returns a 'memoryArea' -
a reference to the actual bytes in flash*/
function fwrite(data) {
  var len = data.length;
  while (data.length&3) data+="\xFF";
  var a = addr;
  flash.write(data, addr);
  addr += data.length;
  return E.memoryArea(a,len);
}

// So write all our data into that flash now...
okey = fwrite( atob("MIIEogIBAAKCAQEAwcBeteXIax0S+gNFv4f0K+q4LOv/E7kmiT8IvZl85NvUDF/slro/x/EreRMGmjcxoeT7+U62nrVu07lZhc+WAiluBX6Ykb2h7lJ/1M3MKV4uK5Yq92p1MxVXzYhVbCKBmcKrPu7Od+hLIMLIMTu0zwc23cE4tSUP4WV38BAzol3k9Rhd1MIjdZ3QUBTYYrvPveUv1TCdsQNsAGv5ZhJx57Anm52W7jTzwBw1O4cn31TD0KD3US/yDKL2IH7BSBFiadN5AzBJbh/ch16/aOfUBIlKNZf9LofMOVLxNm9hxpP2vgckl9u9nTtWiCR5EmKZaTqTqN7bsoLkNKR1GJ6yBwIBIwKCAQAQm3XVE7IXz0rE+PdZj09xeoTQpoOT+e1cOJpZZO1ys8G36vcFmu+GKp1ThUm1cnH37w5Ikbfht8erv24SyKZ1NsBJnSMFLYLoirp/9GH06tDB6EzTYOV5aDq52HxZudCYJqD2xAMRl5FpNURb/c2rWGPJ3Vyhz/oMAWINQzegM8in120G0ZmEnbdvdDWivWKJ4Ap1/fqiWr6yBLbG7gKnLkU3PksQG0YtTahSGXdfEPAi7+CIcRi8svM2Agecm7DH3MW/pgHY8JX87psAmgqCONyVqcAiBlBwQrIk59pczKuzCOSp66STM5S7pXwIuWz/YDJPaGPIL8U/GAyQvJxjAoGBAPfc71FJNg+pKvQMUybGutDtACkexYBNVInpkAKBtGszPjN8xVmmh9huHj+zooFSxQPB2FmsNW7i69pf09FFFgf2tDL22BcG9VldpJR8l77EVw2arovoKC1TyCWxNvD+LYA6VIdd0OXUfUIXG9RGnnnNmHplO5IQ4ka2KI3XEuBtAoGBAMgcrRSRVBTBQtWEyz/cCivdrzdV2AaoNtv5+k0o2pJrViHvrCbhGZNBc/R0u4s3dM53L2IX0w065LD1PeyKjaFXZO2lx3Iu3lf4L8U6oEVQ8C8eLQUOwJ8dK1fWp304Z/iwStwmTa0TZE05KKhl+31NlSOlNr2CHfWeY1So81vDAoGBAKn2lXmRSaRWvl40VkZ5pKyFQfBPnV9K+CQN3xepZcXZ/sQ4TM/CpktEMf/LoqHSWzXGwD19ZnfsD3ErxHI+AHp9SF12EITRkkvocNrZF5jBJcAvjaHDw8dPZKwhv0YqotuVtk4xs9DMOKJY/SPYpy7zYT38RhsEQ2OwG8754Q7rAoGASlPLQidZvpDs8DijQ5rfNN1PtXeoAnj+bvZy6XWTAy8unuP+HRHHq7k5spHCAIJP9Opwr2fvTg6PdO1gJKh9v5V9QlOEmCAJcSGrV+KTTPItU1RZ3U6fUQrVlaeACfBhIdsUfafTtVBYdHRQ7hdAJzoS0rm1PxMScSwzhdhaY+kCgYEAtpWlQzz7u+lurGKbKZY1cTPiFprA9PIsCtl0A5PdQ1XPnYfBgVAlB26oF9ubNy8VxpoerT7VlfILOSBRHx4n2VaUe/7iyZX4yNC+J8zKyLLq8ea0GC76d/masFMuWvZcuOk0afnBZ4VffGUxYMdjP8CEWFJjYjJpsWzW3uWB2ek="));
ocert = fwrite( atob("MIIFijCCA3KgAwIBAgIJAJhrfxuduldTMA0GCSqGSIb3DQEBCwUAMIGBMQswCQYDVQQGEwJVUzELMAkGA1UECBMCTUExDzANBgNVBAcTBkJvc3RvbjETMBEGA1UEChMKRXhhbXBsZSBDbzEQMA4GA1UECxMHdGVjaG9wczELMAkGA1UEAxMCY2ExIDAeBgkqhkiG9w0BCQEWEWNlcnRzQGV4YW1wbGUuY29tMB4XDTE1MTEyNzEyMjM1NloXDTE4MDgyMjEyMjM1NlowgYYxCzAJBgNVBAYTAlVTMQswCQYDVQQIEwJNQTEPMA0GA1UEBxMGQm9zdG9uMRMwEQYDVQQKEwpFeGFtcGxlIENvMRAwDgYDVQQLEwd0ZWNob3BzMRAwDgYDVQQDEwdjbGllbnQxMSAwHgYJKoZIhvcNAQkBFhFjZXJ0c0BleGFtcGxlLmNvbTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBANEmBWwVwLrRcIzF2Juu45tfP0CO88/80QlBMZPPAT7HbIo1YLeBGy3d12LxywT8/1zGlsxxA5xJS1wumUE3xIOAZRZG++6JshstfNPvUNbmIFNNvr3cKpppWX7rPqK39/cKniBaoLLl6RaYrIuBDcgL1Pofs5XbdsPk92fn82KyDrpyYzuI6KNjWMTtyoIDiC8vftY9JeL+IMAVQ3s+oNdxQiE1sU252BSDVXymGxoKDs8+EpbCnTkU8HdffZ4o7rWFEj9FzM/PgZmNF9c2YLsLLlh3QmMxZhfXdMmElgfCbK54uKsujFFxQI+whX2gwy1qeWkJBywSpj7g4SDizIN8jXetT5J4r/zSURKwlMeboZdUd5fs1us4JtN0Ba2D7Tch+cKenyM/iRo3HPVXvsL2FA4cuEdEwOM8kADeZqIO9yrfp3rNsKkEY/a8e96jLniRx1DD9Csk8xvXk95UBTAuSZeg3oGFPBALa6XK3PKLd1EHm4un9DO2TvgaypfSIudz0hXqapOavxz2IyOyqigpJqDR9fXe9WKa3oD0fwS5SgBmcjmy/73JTDDiv4fgCztLPZgSftMPNBy3HDJxyf1uRVyOOMebL4jfxVrgM/kIzMSz4YSMGazKLknKz9x6PtEmjEeVlJNAmoXmT6zXA9N+4+kUanG6XE2IrvD+MfnrAgMBAAEwDQYJKoZIhvcNAQELBQADggIBAAPv3w4KVca2vZeaPN4kHb7ln1ZkXimZ/jZYJMdFh0xcwnTgGQiW+P2voIJuA1GsrrdLvD27RnV1UKtDbJT+MZB5nM/mt7BMyQKdHEGy1YFFLFQz4YMaUEoif5OXFFnmunEu76C90qwbxBpUiS3lB97Gipy3VDBJKFE2kYaypYJc0XqIJcnypzsLBU/K9Bl13Xvj7QNN2VyqDGKlw6v6UJWRyYT7efqvvJ5Ljglmdn1UxX+WmfLzKtO+aMBoSuOgyFEttLLKESYXYRRcomfCRxqIH3XA3PzyDEN5R/wG38IQD3Y0Zt+UYabS6qUKtD2jMD8dL1gr7NWLraDSPAre2fBbHtjskr8vyR5PjrBFLJWOzEQKzclxW3O6cmZKyjwd092JuNn+FSjgo/glWik8jyFXJzK5bLXgGFa31YFnrmWKDrxbAWCuJL6UcRx4rX9qdPkZpwPTqN1sEh1YqdZShxDTFjbxDrE8gL5xbo62q9bdzbN/TMzhVo1BYvQytt7MbX2ZEXDXOup2QiOs23MqcQsf3yjT25OD5V9w3NWXDcd+TLsNCdKFnY+EpWOe4qs7k4UuXJMcW/zAPPBZDPEBsi+AAYsNYEo8QdsCcNtiWD818fTjHR6nmNRsMjRH9jeM9x0N/fJvsuCrrMQZF5KNpntOP0lV1ktAIcjQJUf93rN+"));
oca = fwrite( atob("MIIFgDCCA2gCCQD1KANs3obrTjANBgkqhkiG9w0BAQsFADCBgTELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAk1BMQ8wDQYDVQQHEwZCb3N0b24xEzARBgNVBAoTCkV4YW1wbGUgQ28xEDAOBgNVBAsTB3RlY2hvcHMxCzAJBgNVBAMTAmNhMSAwHgYJKoZIhvcNAQkBFhFjZXJ0c0BleGFtcGxlLmNvbTAeFw0xNTExMjcxMjIyMzFaFw00MzA0MTMxMjIyMzFaMIGBMQswCQYDVQQGEwJVUzELMAkGA1UECBMCTUExDzANBgNVBAcTBkJvc3RvbjETMBEGA1UEChMKRXhhbXBsZSBDbzEQMA4GA1UECxMHdGVjaG9wczELMAkGA1UEAxMCY2ExIDAeBgkqhkiG9w0BCQEWEWNlcnRzQGV4YW1wbGUuY29tMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAwjlJ3eyrnGIiJiplY5mvIaaoMC20oR4Jx5+FOXNJJhjSz9mOoqcpEe2U6ZmmVfcpc8zdt1f9KkED5yjzpnF02CG+KYaRs/Rfj0NJejcpBN3Hn4R6+yJek2rYrKi4uZyMZrWx/8PTp/lEypAEBf/2vX9WYNgi5eWyHeHEfJ4yucRI1UgRw5W/RLYoAePAPT1ekB66NbosBIZhRXJvjqED/jOMOlpHgNPQHTUyPj5lR8ZTSduATUQac6qRx9pdYyVICVE8bxma56R0pX8Cdx/wg+5gKOXUPuXW1xTPNuH4JpNTd7huJwa8Ff0ReKTLHZk1hYB7uDzL5moc8kylYEwz2W13KyXxZbh3kClVSNZvPbrcp1eWoyJyznLRLVv3cCWQwmEMr1071Th4/6dnUZ/wl085HIjV93X9H8nzZ7VBkwHD8ZHB1foW0/jgbK0qsH2gV7frGFPxZydmk/Nrwdl3RtoM+xEpQI7cPYDf9j/CX/ynM4LfOeHhVfMrx0I3E8wNPX9MU+O+Pu6wsT4WbGkNhcuS4oE2As13obMnUebLwOxxWu4ErI8WG1ITwRVEQik72Iqj0d5vvjZw2z5TFzkMB/Tl3qNvAp9jzpNtvRUyrS6KRY667BZOJ9TJK/5+jenGEi7+UNq9ig490WnYDBPA/N9QSPHCx6TLNapayQvem5UCAwEAATANBgkqhkiG9w0BAQsFAAOCAgEAS9mpX4QgnwntvH9wutY+zOWBLejd/psjjVmZdYzeVC6fkCaw0Qj1unszq58EHlA8275ARTYyicRHIYLF3ZDYwMxUCu7iIQJUzVYJRqowV2Ap4OeMlh5sUb/Wlmhs5TauSZ1gz4LqMqnGkVMrvvU+1WX2ePFl81nnr1UMb2+dLzC0Gj2jH3tlzW76yFD4gElR8W4ypgvAw9pFKlSuOc4y6KS32jaOJk9zWSigPQadI1pOSSk+iKuPp98BBY/gDw+FBKUNARo3ci5F2s1dJZwQSCnuBhVb1r+3aHjnp4PAO7Mq7YyFf7qwSmuA++nF7TGhu3lf3hY1Jzgo+rOi+pFZtebCsewLDJjyNUxhQkOIf4TutC9wPJUtCKDpLA0iJso1AX5297iap5g7y0J0fUls16U+F0arsBHhgIN5ARifImeE+1bVVx7kLtklA23njczUt88ylgsCEnYyu0U+0+kwAkMDyPWDd3KPfDhykhQJ0Ev/44+JTDk3zydO2YvCpqMqLuhbZr9mvmV7uzJVbFnLr3sU75upv0N4JQLIn3XyTtDbWjTqA8d6qbG5BxYL5xkrWRxo3Fd6r1AyFje1ilKUkwZ1YZZCDvgv/2uEFyL0VlgfHfBhNbN2nsTTxijKc19ROeIRXl7eqWq/cyIbA/5LzEy2UbbEhKIroPGosQML4sc="));


// Actually send a form
function sendForm() {
  // This uploads to https://docs.google.com/spreadsheets/d/1R1D6GKK5MvtjS-PDEPHdqIqKSYX4kG4dPHkj_lkL1P0/pubhtml
  LED1.set(); // light red LED while we're working
  var content = "entry.1093163892="+encodeURIComponent(E.getTemperature());
  var options = {
    host: 'docs.google.com',
    port: '443',
    path:'/forms/d/1bBV4map47MPRWfaHYCEd1ByR4f_sm3LSd3oRdYkiVKg/formResponse',
    protocol: "https:",
    method:'POST',
    headers: { 
      "Content-Type":"application/x-www-form-urlencoded",
      "Content-Length":content.length
    },
    key : okey,
    ca : oca,
    cert : ocert
  };
  
  console.log("Connecting to Google");
  require("http").request(options, function(res)  {
    console.log("Connected to Google");
    var nRecv = 0;
    res.on('data', function(data) { nRecv += data.length; });
    res.on('close', function(data) {
      LED1.reset(); // turn red LED off when finished
      console.log("Google connection closed, "+nRecv+" bytes received");
    });
  }).end(content);
}



function onInit() {
  clearInterval();
  // initialise the ESP8266, after a delay
  setTimeout(function() {
    digitalWrite(B9,1); // enable on Pico Shim V2
    Serial2.setup(115200, { rx: A3, tx : A2 });
    var wifi = require("ESP8266WiFi_0v25").connect(Serial2, function(err) {  
      if (err) throw err;
      console.log("Connecting to WiFi");
      wifi.connect("Espruino","helloworld", function(err) {
        if (err) throw err;
        wifi.getIP(function(e,ip) {
          LED2.set();
          console.log(ip);
          setInterval(sendForm, 60000); // once a minute
        });
      });
    });
  }, 2000); 
}
```

* Now go to `drive.google.com` (you need to be logged in with your Google account)
* Click `New`, `more`, `Google Forms`
* Edit the title
* Add a First item with title `Temperature` and set the question type to `Text`
* Click `Done`
* Click `Send form` and copy the URL you're given
* Open it in the web browser, right-click and `view source`
* Search for `<form` and copy the `/forms/...` but out of the `action` parameter and paste it into `path` in `sendForm`
* Search for `<input type="text"` and copy the `name` parameter out and paste it into where it says `var content =` in `sendForm` - make sure there is an `=` on the end.
* Now upload the code to Espruino, and type `save()` to save it to flash - it should now start running.

What now?
---------

### Logging more data

Logging more data is easy - you can just add more items to your form and can append extra items to the `content` variable. The content is just like a URL... To add more items, you need to separate them with an `&` character.

### Changing the encryption key

You should use your own encryption keys as the ones here are now in the public domain. See [the `tls.connect` page](/Reference#l_tls_connect) for more information on how to do this.
