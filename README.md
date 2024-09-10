# asterisks2Fa

Original from 2012. :) Still works! I'll post my Go version of the PHP code later.

I was playing around with trying to implement a one-time password solution and thought how some people may not have a cell phone or SMS service to receive the code. Instead, I wanted to try a solution that would speak a code or the user could select to use SMS, if they have that service available. Voice-enabled two-factor auth seemed to be only available as a commercial product. I'm developing an app that is open source so I don't want the user of the app to have to purchase anything and I'm a po' man's son. :^) I decided to see if I could do it with an open source solution and since I've used Asterisk for a few years and familiar with it, I looked into it for a solution, and found one.

I used the Asterisk Call Files feature. When a properly formatted file is placed into the "outgoing" directory, Asterisk will read it and execute the commands. The documentation is available here with more examples: http://www.voip-info.org/wiki/view/Asterisk+auto-dial+out

I am developing my application in PHP so I used its native functions to create a code, pull the phone number from the database, and create a formatted .call file with those values and moved it into the /var/spool/asterisk/outgoing directory and the call was placed and the code read to the user. If the user supplies the proper code, after their username and password has succeeded, then they are taken to the main application dashboard.

You will need to create a form to authenticate via username and password and the redirect to the PHP code below.

.call file (this is the minimum needed for this to work)

```
channel: local/NUMBER_TO_CALL@from-internal
callerid: MY_VOIP_NUMBER
application: Background
data: silence/2&en/please-enter-the&digits/1&en/time&vm-password&silence/1&digits/2&silence/1&digits/1&silence/1&digits/4&silence/1&digits/9&silence/1&digits/9&silence/2&en/your&digits/1&en/time&vm-password&en/is&silence/1&digits/2&digits/1&digits/4&digits/9&digits/9&en/goodbye
```
Note that the paths of the sound files are relative to the /var/lib/asterisks/sounds for my setup. Accordingly, the file vm-password is located at /var/lib/asterisk/sound/vm-password.wav while the file digits/1 is located at /var/lib/asterisk/sound/digits/1.wav. Also, note that the file extesion is not added.

Readable format below

<table border="1">
<tbody><tr>
<td bgcolor="#EAF7FB"><b>Speaker</b></td><td bgcolor="#EAF7FB" align="center" colspan="9"># Pause for 2 seconds to give the user time to answer the call.<br>
<b>silence/2</b></td>
</tr>

<tr><td bgcolor="EAF7FB" colspan="10"># Concatenated sound files.  I cheated to get the speaker to say "one" (digits/1) and "password" (using the vm-password sound file. :^)</td>
</tr>
<tr>
<td bgcolor="#EAF7FB"><b>Speaker</b></td><td align="center" bgcolor="#EAF7FB">Please enter the</td> <td align="center" bgcolor="#EAF7FB"> one </td> <td align="center" bgcolor="#EAF7FB">  time </td> <td align="center" bgcolor="#EAF7FB" colspan="6">password</td>
</tr>
<tr>
<td><i>Asterisk File</i></td><td align="center"> en/please-enter-the </td>  <td align="center">digits/1</td>  <td align="center"> en/time</td> <td align="center" colspan="6"> vm-password</td>
</tr>

<tr>
<td bgcolor="#EAF7FB"><b>Speaker</b></td><td align="center" bgcolor="#EAF7FB" colspan="9"># Give the user a second to get ready to enter the code.<br>
<b>silence/1</b></td>
</tr>

<tr>

<td bgcolor="#EAF7FB"><b>Speaker</b></td><td align="center" bgcolor="#EAF7FB">2</td><td align="center" bgcolor="#EAF7FB">Pause</td><td align="center" bgcolor="#EAF7FB">1</td><td align="center" bgcolor="#EAF7FB">Pause</td><td align="center" bgcolor="#EAF7FB">4</td><td align="center" bgcolor="#EAF7FB">Pause</td><td align="center" bgcolor="#EAF7FB">9</td><td align="center" bgcolor="#EAF7FB">Pause</td><td align="center" bgcolor="#EAF7FB">9</td>
</tr><tr>
</tr>
<tr><td><i>Asterisk File</i></td><td align="center">digits/2</td> <td align="center"> silence/1</td> <td align="center">digits/1</td><td align="center">silence/1</td><td align="center"> digits/4</td><td align="center"> silence/1</td><td align="center">digits/9</td> <td align="center">silence/1</td><td align="center">digits/9</td>

</tr>

<tr>
<td bgcolor="#EAF7FB"><b>Speaker</b></td><td align="center" bgcolor="#EAF7FB">Pause </td>   <td align="center" bgcolor="#EAF7FB">Your</td>    <td align="center" bgcolor="#EAF7FB">One</td> <td align="center" bgcolor="#EAF7FB">time</td> <td align="center" bgcolor="#EAF7FB">password</td> <td align="center" bgcolor="#EAF7FB" colspan="4">is</td>
</tr>
<tr>
<td><i>Asterisk File</i></td><td align="center">silence/2</td> <td align="center">en/your</td> <td align="center">digits/1</td>  <td align="center"> en/time</td> <td align="center">vm-password</td>  <td align="center" colspan="4"> en/is</td>
</tr>
<tr>
<td bgcolor="#EAF7FB"><b>Speaker</b></td><td align="center" bgcolor="#EAF7FB">Pause</td>          <td align="center" bgcolor="#EAF7FB">2</td>                   <td align="center" bgcolor="#EAF7FB">1</td>                         <td align="center" bgcolor="#EAF7FB">4</td>                 <td align="center" bgcolor="#EAF7FB">9</td>                <td align="center" bgcolor="#EAF7FB">9</td>               <td align="center" bgcolor="#EAF7FB" colspan="4">Goodbye</td>
</tr>
<tr>
<td><i>Asterisk File</i></td><td align="center"> silence/1</td>  <td align="center"> digits/2</td>   <td align="center">digits/1</td>  <td align="center"> digits/4</td>  <td align="center">digits/9</td>   <td align="center">digits/9</td>   <td align="center" colspan="4">en/goodbye</td>
</tr>
</tbody></table></center>
I hope the table makes it easier to read. I removed the ampersands (&) for readability, but it is required in the .call file to play each sound file in sequence. These are just sound files from the Asterisk sound files directory (/var/lib/asterisk/sounds) played in sequence and separated by the ampersand. I had to download the extra sound files from Asterisk to get the "digits" so those could be read back (http://www.asterisk.org/index.php?menu=download). In the example above, the user will hear:

"Please enter your one time password 2 1 4 9 9. Your one time password is 2 1 4 9 9. Goodbye"

below is a sample of my PHP code that creates the random code, reads each digit from the random code, creates the .call file and places the call. This is all on a development system and the Asterisk server only makes outbound calls and doesn't accept incoming calls and no ports opened to allow external access to it. I changed the group permissions on the /var/spool/asterisk/outgoing so that it was owned by the webserver user. Then I gave the group "write" permissions so it could move the .call file into the outgoing directory.

After that, it worked as expected.

	
	      // Generate a 5 digit number
        // http://elementdesignllc.com/2011/06/generate-random-10-digit-number-in-php/
        $random = substr(number_format(time() * rand(),0,'',''),0,5);

        // Set as a session variable to be passed to the check above after the user submits the code.
        $_SESSION['s_codeToEnter'] = $random;

        // The $number is the user who will be called. This will need to come from some other form or backend database.
        $number = $_SESSION['s_number'];

        // Random digits to put into the call file
        $num1 = $random[0];
        $num2 = $random[1];
        $num3 = $random[2];
        $num4 = $random[3];
        $num5 = $random[4];

        // Callout file
        $call_data = "local/$number@from-internal\n";
        $call_data .= "callerid: MY_VOIP_NUMBER\n";
        $call_data .= "application: Background\n";
        
        // Reads "Please enter your one time password  .  Your one time password is . Goodbye" then hangs up the call.
        $call_data .= "data: silence/2&en/please-enter-the&digits/1&en/time&vm-password&silence/1&digits/$num1&silence/1&digits/$num2&silence/1&digits/$num3&silence/1&digits/$num4&silence/1&digits/$num5&silence/2&en/your&digits/1&en/time&vm-password&en/is&silence/1&digits/$num1&digits/$num2&digits/$num3&digits/$num4&digits/$num5&en/goodbye";

        // Generate a 15 digit number for the file.
        $rand_file = md5(substr(number_format(time() * rand(),0,'',''),0,15));

        // write the callout file
        $myFile = "/tmp/$rand_file.call";
	$fh = fopen($myFile, 'w')
          or die("Unable to process one-time code.");
        fwrite($fh, $call_data);
        fclose($fh);

        $old = "/tmp/$rand_file.call";
        $new = "/var/spool/asterisk/outgoing/$rand_file.call";
	// move to the outgoing directory
        rename($old, $new)
          or die("Unable to process one-time code at this time. Please try again later.");

        // Emptying these variables to be on the safe side.
        $random = "";
        $rand_file = "";
        $myFile = "";
        $old = "";
        $new = "";
        $call_data = "";
        $num1 = "";
        $num2 = "";
        $num3 = "";
        $num4 = "";
        $num5 = "";

        <b> Your security code has been sent and you will receive a call (there will be a two second delay before hearing the code).  Please enter the code in the box below.</b><p>
        <form method="post" action="">

                One-time code: <input type="text" size="6" name="code">  <input type="submit" name="submit" value="Submit">

        <form>

At the top of the script is a check to see if the proper code was entered after the user submitted the code.

    if ($_POST['submit'] == "Submit") {

        $code = $_POST['code'];

        if (!preg_match("/^[0-9]{5}$/", $code))
          die("Invalid code.");

        if ($code == $_SESSION['s_codeToEnter']) {

                // Prevent multiple auths in case the user returns to this page and has already authenticated again.
                $_SESSION['s_preauth'] = "HAS_PREAUTH";

                header('Location: msg_main.php');
                $_SESSION['s_codeToEnter'] = "";
                exit();

        } else {

                echo "Invalid Code Entered.";
                exit();

        }

The user supplies their username and password in a separate form. If that succeeds, then they get to the page that generates their code and calls them.

In production, the Asterisk server will reside on a separate server and an application will monitor the directory where Asterisk creates the .call file and transfer it to the remote Asterisk server.

That's it. I hope someone finds this useful.

## Full example:


    if ($_POST['submit'] == "Submit") {

        $code = $_POST['code'];

        if (!preg_match("/^[0-9]{5}$/", $code))
          die("Invalid code.");

        if ($code == $_SESSION['s_codeToEnter']) {

                // Prevent multiple auths in case the user returns to this page and has already authenticated again.
                $_SESSION['s_preauth'] = "HAS_PREAUTH";

                header('Location: msg_main.php');
                $_SESSION['s_codeToEnter'] = "";
                exit();

        } else {

                echo "Invalid Code Entered.";
                exit();

        }

    } else {
    
        // Generate a 5 digit number
        // http://elementdesignllc.com/2011/06/generate-random-10-digit-number-in-php/
        $random = substr(number_format(time() * rand(),0,'',''),0,5);

        // Set as a session variable to be passed to the check above after the user submits the code.
        $_SESSION['s_codeToEnter'] = $random;

        // The $number is the user who will be called. This will need to come from some other form or backend database.
        $number = $_SESSION['s_number'];

        // Random digits to put into the call file
        $num1 = $random[0];
        $num2 = $random[1];
        $num3 = $random[2];
        $num4 = $random[3];
        $num5 = $random[4];

        // Callout file
        $call_data = "local/$number@from-internal\n";
        $call_data .= "callerid: MY_VOIP_NUMBER\n";
        $call_data .= "application: Background\n";
        
        // Reads "Please enter your one time password  .  Your one time password is . Goodbye" then hangs up the call.
        $call_data .= "data: silence/2&en/please-enter-the&digits/1&en/time&vm-password&silence/1&digits/$num1&silence/1&digits/$num2&silence/1&digits/$num3&silence/1&digits/$num4&silence/1&digits/$num5&silence/2&en/your&digits/1&en/time&vm-password&en/is&silence/1&digits/$num1&digits/$num2&digits/$num3&digits/$num4&digits/$num5&en/goodbye";

        // Generate a 15 digit number for the file.
        $rand_file = md5(substr(number_format(time() * rand(),0,'',''),0,15));

        // write the callout file
        $myFile = "/tmp/$rand_file.call";
	      $fh = fopen($myFile, 'w')
          or die("Unable to process one-time code.");
        fwrite($fh, $call_data);
        fclose($fh);

        $old = "/tmp/$rand_file.call";
        $new = "/var/spool/asterisk/outgoing/$rand_file.call";
	      // move to the outgoing directory
        rename($old, $new)
          or die("Unable to process one-time code at this time. Please try again later.");

        // Emptying these variables to be on the safe side.
        $random = "";
        $rand_file = "";
        $myFile = "";
        $old = "";
        $new = "";
        $call_data = "";
        $num1 = "";
        $num2 = "";
        $num3 = "";
        $num4 = "";
        $num5 = "";
    
    ?>

        <b> Your security code has been sent and you will receive a call (there will be a two second delay before hearing the code).  Please enter the code in the box below.</b> 
        <p>
        <form method="post" action="">

                One-time code: <input type="text" size="6" name="code">  <input type="submit" name="submit" value="Submit">

        <form>
      
      <?php
      
      }
      
      ?>
