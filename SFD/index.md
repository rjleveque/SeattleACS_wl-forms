# Winlink Form for SFD Incident Card

Scroll down on [the repository README
page](https://github.com/rjleveque/SeattleACS_wl-forms/tree/main/SFD)
for background information and some screen shots.

## Try it out

[Click here](https://rjleveque.github.io/SeattleACS_wl-forms/SFD/SFD_incident_card-form.html)
to open an html form in your browser that you can try filling out.

In this version the "Submit" and "Load Data" buttons do not work, but you
can "Save Data" and it should create a small JSON (JavaScript Object Notation)
data file and save it to your
own computer with a filename based on the date and time.  Later you could
upload one or more of these to a computer running Winlink for submission.

## Installing as a custom form in Winlink

To use this form in Winlink, you need to copy two files to the
proper directory\
 [`RMS Express\Global Folders\Templates`]:

- `SFD_incident_card-form.html` is the html form that you see when you
  compose a message.  (This is the same file that is opened on a website
  if you test this with the link above.)

- `SFD_incident_card-template.txt` is a file that tells Winlink how to
  compose a Winlink message based on the data in the form.

Eventually a third file will also be included:

- `SFD_incident_card-viewer.html` will be used to take an *incoming* Winlink
  message using this form and reformat it as a nice webpage for viewing.
  Even without this form, you can see all the information in the main body
  of the incoming Winlink message.

You can download these files from the [Github
repository](https://github.com/rjleveque/SeattleACS_wl-forms/tree/main/SFD)
for testing, but they are not fully functional yet.
Versions for members to use will be provided later.


## Save the html file on a laptop, tablet, or cell phone

Experimenting with the test form
linked above requires an internet connection to the
computer where the html file it is hosted.

For field applications, you might want to have this html file on your own
laptop, tablet, or cell phone so that you can input data and save the files
even if you don't have Winlink (e.g. on a Mac or iOS).  The resulting JSON
files can then
be transferred later (e.g. using bluetooth or a USB thumbdrive)
to a different computer for submission into Winlink and sending. 

This could be useful for radio operators who are out on a route with SFD and
do not have Winlink access until they return to the station, or for operators
at a station who are dealing with many cards, so that the computer hooked to
the radio for transmission is not bogged down by the need to type in the data.

The html form could also potentially be used directly by SFD personnel to
input data in the field for later transfer to a radio operator.

### RadioMail

The RadioMail app for iOS can be used to do Winlink on an iPhone (you also
need a TNC to connect to the phone by bluetooth and appropriate cable to
connect it to your radio, or you can use Winlink Telnet, e.g. if you have
HamWAN access). The app
costs a bit and then you also need a subscription if you want to use forms
on it.  If so, it has all the standard form (e.g. ICS forms) as well as
Washington State forms, and you can install custom forms. See
the [RadioMail website](https://radiomail.app/help/forms) for more
information.

## The code

The code is in this Github repository:
[github.com/rjleveque/SeattleACS_wl-forms](https://github.com/rjleveque/SeattleACS_wl-forms/)

The `.html` and `.txt` files are also in [this zip
file](https://faculty.washington.edu/rjl/hamradio/SFD_incident_card-form.zip).
You are welcome to test it, but note that this is still Work in Progress,
and not yet intended for general distribution to the ACS membership.
Better versions and more information on installation will be distributed
later.

If you have problems with it, or have suggestions, you are welcome to [raise an
issue](https://github.com/rjleveque/SeattleACS_wl-forms/issues) (which
requires a free Github account) or send email to Randy KJ7WLT.

If you want to help develop it further, feel free to reach out or modify the
code and submit a pull request.

The forms would benefit from more work by someone with javascript
experience; this is not my native language!

