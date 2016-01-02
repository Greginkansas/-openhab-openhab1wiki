# Introduction

Follow the instructions here to setup your own TTS service running on a local machine. This is ideal for use with the Google TTS or Squeezebox action bundles, and removes the requirements for an internet connected service and the inherent delays with having to send such requests out to the cloud and wait for a response.

You will need a web server configured with PHP but all other steps needed should be below.


# Installation

* install the necessary dependencies (pico2wave, lame)
  - `sudo apt-get install libttspico0 libttspico-utils libttspico-dev libttspico-data lame`
* install PHP composer (needed to install/use phplame)
  - `curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`
  - see https://getcomposer.org/doc/00-intro.md for more details
* copy the contents of the two files (composer.json and tts.php) below to a folder under /var/www
  - e.g. /var/www/tts
  - ensure your web server has sufficient permissions to create files in this directory (e.g. owned by www-data)
* run composer to resolve the necessary dependencies
  - `sudo composer install`
* setup your web server to allow requests to /var/www/tts
  - beyond the scope of these instructions
* test by typing in the following to your browser;
  - http://host/tts/tts.php?lng=en-GB&msg=Hello

# Dependencies

* pico2wave
  - `sudo apt-get install libttspico0 libttspico-utils libttspico-dev libttspico-data`
  - `pico2wave -l=en-GB -w=announce.wav "The coffee machine is ready"`
  - example of Python script using it - https://github.com/the-kyle/picospeaker
* lame
  - `sudo apt-get install lame`
  - `lame announce.wav announce.mp3`
* phplame
  - call lame from a PHP script
  - https://github.com/b-b3rn4rd/phplame
* php serve mp3
  - http://us2.php.net/manual/en/function.readfile.php
  - http://stackoverflow.com/questions/1516661/can-i-serve-mp3-files-with-php
* squeezy (unused but useful for manually sending URLs to Squeezeserver)
  - CLI interface to Squeezeserver
  - https://github.com/pssc/squeezy

### composer.json
```
{
    "require": {
        "b-b3rn4rd/phplame": "dev-master"
    }
}
```
### tts.php
```
<?php

$lng     = $_GET['lng'];
$msg     = $_GET['msg'];

$filewav = "tts.wav";
$filemp3 = "tts.mp3";

require 'vendor/autoload.php';

use Lame\Lame;
use Lame\Settings;

// perform TTS using pico2wave
try {
    exec('/usr/bin/pico2wave -l=' . $lng . ' -w=' . $filewav . ' "' . $msg . '"');
} catch(\RuntimeException $e) {
    var_dump($e->getMessage());
}

// encoding type
$encoding = new Settings\Encoding\Preset();
$encoding->setType(Settings\Encoding\Preset::TYPE_STANDARD);

// lame settings
$settings = new Settings\Settings($encoding);

// lame wrapper
$lame = new Lame('/usr/bin/lame', $settings);

// convert wav - mp3 using lame
try {
    $lame->encode($filewav, $filemp3);
} catch(\RuntimeException $e) {
    var_dump($e->getMessage());
}

// send the converted mp3 back to the client
if(file_exists($filemp3)) {
    header('Content-Transfer-Encoding: binary');
    header('Content-Type: audio/mpeg');
    header('Content-length: ' . filesize($filemp3));
    header('Content-Disposition: filename=' . $filemp3);
    header('Cache-Control: no-cache');
    header('icy-br: 64');
    header('icy-name: TTS Announcement');
    readfile($filemp3);
} else {
    header("HTTP/1.0 404 Not Found");
}

?>
```