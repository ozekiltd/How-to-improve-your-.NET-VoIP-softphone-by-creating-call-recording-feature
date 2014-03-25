#How to improve your .NET VoIP softphone by creating call recording feature in C#

**It describes how to extend your softphone by building call recording funcionality in C#. You can get to know how to record and store your phone conversations in a .wav file.**

##Introduction

Due to their easy usage and easy expandability with different features, softphones are really popular, especially among call centers. One of these features is call recording. I am going to show you in this article how you can implement it in your own .NET VoIP softphone application. After studying this guide, you will be able to save the conversations made with your softphone in .wav format.

##Background

I have used Ozeki VoIP SIP SDK for implementing the call recording feature in my softphone, because it has numerous prewritten VoIP components that I needed. I have written the application in C# with Microsoft Visual Studio 2010 that needs at least .NET Framework 3.5.
Before describing the project I thought I will give an example application for those readers who don’t have their own softphones. You can implement the call recording in it.

You can download the example application from this link:

http://voip-sip-sdk.com/p_136-c-sharp-windows-forms-softphone-voip.html

##Implementing the call recording feature

If you have a working softphone, you need to add some extra functionalities to it. After you have added them the call recording will be done automatically.

You can record the incoming and outgoing voices separately in two different .wav files or together in one. In either way you will need the WaveStreamRecorder media handler object to capture the audio into .wav files. 

As I mentioned earlier if you want to capture both the incoming and outgoing voice in one file you will need an AudioMixerMediaHandler. It is going to mix the incoming and outgoing voices together. You can see the implementation of these two objects below.

```
WaveStreamRecorder recorder;  
AudioMixerMediaHandler mixer = new AudioMixerMediaHandler();  
  
string filename;  
string caller; 
```
**Code example 1 – WaveStreamRecorder and AudioMixerMediaHandler**

You can see two strings below the objects, these specify the name of the recorded file so, that the filename will be the parameter for the recorder’s constructor. The caller will be the dialled number in case of an outgoing call or the displayed name of the actual caller in case of an incoming call.

In case of an outgoing call you will have to put the caller into the Pick Up button’s event handler as you can see below.

```
private void PickUp_Click(object sender, EventArgs e)
        {
            MessageBox.Show("Please note, that all your calls made by this softphone will be recorded.", "", MessageBoxButtons.OK, MessageBoxIcon.Information);
            if (inComingCall)
            {
                inComingCall = false;
                call.Accept();
                return;
            }
	
            if (call != null)
                return;

            if (string.IsNullOrEmpty(DialedNumber.Text))
                return;

            if (phoneLineInformation != PhoneLineState.RegistrationSucceeded && phoneLineInformation != PhoneLineState.NoRegNeeded)
            {
                MessageBox.Show("Phone line state is not valid!");
                return;
            }
            caller = DialedNumber.Text;
            call = softPhone.CreateCallObject(phoneLine, DialedNumber.Text);
            WireUpCallEvents();
            call.Start();
        }
```
**Code example 2 – Initializing the caller in case of an outgoing call**

In case of an incoming call you will have to put the caller into the softPhone_IncomingCall() method.

```
        private void softPhone_IncomingCall(object sender, VoIPEventArgs<IPhoneCall> e)
        {
            InvokeGUIThread(() =>
            {
                RegistrationState.Text = "Incoming call";
                DialedNumber.Text = String.Format("from {0}", e.Item.DialInfo);
                caller = e.Item.DialInfo.DisplayName;
                call = e.Item;
                WireUpCallEvents();
                inComingCall = true;
            });
        }
```
**Code example 3 – Initializing the caller in case of an incoming call**

When the call state becomes Answered, the application has to start the call recording. The recorder is going to be initialized with the filename that includes the caller string and the current time. The current time is given in the form of hours, minutes and seconds.

The .wav file is going to be stored in the folder of the exe of the application, because there is no path specified for it.

Connect the microphone and the mediaReceiver objects into the mixer then connect the mixer to the recorder. This way the mixed audio stream is going to be recorded into the file that is specified by the filename.

```
                case CallState.Answered:
                    if (microphone != null)
                        microphone.Start();
                    connector.Connect(microphone, mediaSender);
                    
                    filename = caller + "-" + DateTime.Now.Hour.ToString()+"-"+ DateTime.Now.Minute.ToString()+"-"+DateTime.Now.Second.ToString()+".wav";
                    recorder = new WaveStreamRecorder(filename);

                    connector.Connect(microphone, mixer);
                    connector.Connect(mediaReceiver, mixer);
                    connector.Connect(mixer, recorder);
                    if (speaker != null)
                       speaker.Start();
                    connector.Connect(mediaReceiver, speaker);

                    mediaSender.AttachToCall(call);
                    mediaReceiver.AttachToCall(call);

                    recorder.StartStreaming();

                    break;
```
**Code example 4 – Initializing the MediaHandler**

The recording will be started by calling the recorder.StartStreaming() method. Of course when the call ends the recording has to stop and the media handlers have to be disconnected too.

```
                case CallState.Completed:
                   recorder.StopStreaming();

                    connector.Disconnect(microphone, mixer);
                    connector.Disconnect(mediaReceiver, mixer);
                    connector.Disconnect(mixer, recorder);
                    if (microphone != null)
                        microphone.Stop();
                    connector.Disconnect(microphone, mediaSender);
                    if (speaker != null)
                        speaker.Stop();
                    connector.Disconnect(mediaReceiver, speaker);

                    mediaSender.Detach();
                    mediaReceiver.Detach();

                    WireDownCallEvents();
                    call = null;

                    InvokeGUIThread(() => { DialedNumber.Text = string.Empty; });
                    break;
                case CallState.Cancelled:
                    WireDownCallEvents();
                    call = null;
                    break;
```
**Code example 5 – Stopping the recording**

The WaveStreamRecorder object is subscribed for the stopped event as you can see it on the following code example.

```
        void WaveStreamRecorder_Stopped(object sender, EventArgs e)
        {
            if (RecorderStopped != null)
                RecorderStopped(this, EventArgs.Empty);
        }

        public event EventHandler<EventArgs> RecorderStopped;
```
**Code example 6 – The record stopping event handler**

It is wise to inform the user of the application every time he/she makes a call that it is going to be recorded. I have made a pop up window for this reason that will appear every time the Pick Up button is pressed.

```
private void PickUp_Click(object sender, EventArgs e)
{
	MessageBox.Show("Please note, that all your calls made by this softphone will be recorded.", "", MessageBoxButtons.OK, MessageBoxIcon.Information);
…
```
**Code example 7 – The pop up window notification**

With this you have successfully implemented the call recording into your softphone. Go ahead make a test call and see how it works.

##Summary

During this article you could learn how to extend your softphone with call recording functionality that can record the incoming and outgoing voices separately or together. You could learn about the basics of call recording and you should have got a glimpse of how Ozeki VoIP SIP SDK works and how it can help you if you are developing VoIP or SIP applications. 

##References

* You can download the free trial of Microsoft Visual Studio from this site: http://msdn.microsoft.com/en-us/vstudio/default.aspx
* You can download the necessary .Net Framework 3.5 from the following page: http://www.microsoft.com/en-us/download/details.aspx?id=21
* Finally you can get the Ozeki VoIP SIP SDK from: http://www.voip-sip-sdk.com/
