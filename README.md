# MidiDroid [![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-MidiDroid-green.svg?style=true)](https://android-arsenal.com/details/1/4195) [![](https://jitpack.io/v/pdrogfer/MidiDroid.svg)](https://jitpack.io/#pdrogfer/MidiDroid)
MIDI library for Android, ready to use in Android Studio projects.

### Description
This Midi library is an update of [Android MIDI Library](https://github.com/LeffelMania/android-midi-lib), by Alex Leffelman.
Now it is packaged as a convenient Android Library, instead of the original Eclipse project. However, it is a stand-alone Java library with no Android-specific dependencies or considerations. It provides an interface to read, manipulate, and write MIDI files. "Playback" is supported as a real-time event dispatch system. This library does NOT include actual audio playback or device interfacing.

### Usage
**Step 1**. Add [JitPack](https://jitpack.io/) to your root build.gradle at the end of repositories:
```
allprojects {
		repositories {
			...
			maven { url "https://jitpack.io" }
		}
	}
```
**Step 2**. Add the dependency in your module build.gradle:
```
dependencies {
        compile 'com.github.pdrogfer:MidiDroid:v1.1'
        // fork version
        implementation 'com.github.LimeVista:MidiDroid:v1.1.1'
}
```
Use it and enjoy. Contributions and pull requests are certainly welcome.

 
### Examples:
----
#### Reading and Writing a MIDI file:
```java
File input = new File("example.mid");
MidiFile midi = new MidiFile(input);

...

File output = new File("output.mid");
midi.writeToFile(output);
```

#### Manipulating a MIDI file's data:
Removing a track:
```java
midi.removeTrack(2);
```

Removing any event that is not a note from track 1:
```java
MidiTrack track = midi.getTracks().get(1);

Iterator<MidiEvent> it = track.getEvents().iterator();
List<MidiEvent> eventsToRemove = new ArrayList<MidiEvent>();

while(it.hasNext())
{
    MidiEvent event = it.next();
    
    if(!(event instanceof NoteOn) && !(event instanceof NoteOff))
    {
        eventsToRemove.add(event);
    }
}

for(MidiEvent event : eventsToRemove)
{
    track.removeEvent(event);
}
```

Reducing the tempo by half:
```java
MidiTrack tempoTrack = midi.getTracks().get(0);
Iterator<MidiEvent> it = tempoTrack.getEvents().iterator();

while(it.hasNext())
{
    MidiEvent event = it.next();
    
    if(event instanceof Tempo)
    {
        Tempo tempoEvent = (Tempo)event;
        tempoEvent.setBpm(tempo.getBpm() / 2);
    }
}
```

#### Composing a new MIDI file:
```java
// 1. Create some MidiTracks
MidiTrack tempoTrack = new MidiTrack();
MidiTrack noteTrack = new MidiTrack();

// 2. Add events to the tracks
// Track 0 is the tempo map
TimeSignature ts = new TimeSignature();
ts.setTimeSignature(4, 4, TimeSignature.DEFAULT_METER, TimeSignature.DEFAULT_DIVISION);

Tempo tempo = new Tempo();
tempo.setBpm(228);

tempoTrack.insertEvent(ts);
tempoTrack.insertEvent(tempo);

// Track 1 will have some notes in it
final int NOTE_COUNT = 80;

for(int i = 0; i < NOTE_COUNT; i++)
{
    int channel = 0;
    int pitch = 1 + i;
    int velocity = 100;
    long tick = i * 480;
    long duration = 120;
    
    noteTrack.insertNote(channel, pitch, velocity, tick, duration);
}

// 3. Create a MidiFile with the tracks we created
List<MidiTrack> tracks = new ArrayList<MidiTrack>();
tracks.add(tempoTrack);
tracks.add(noteTrack);

MidiFile midi = new MidiFile(MidiFile.DEFAULT_RESOLUTION, tracks);

// 4. Write the MIDI data to a file
File output = new File("exampleout.mid");
try
{
    midi.writeToFile(output);
}
catch(IOException e)
{
    System.err.println(e);
}
```

#### Listening for and processing MIDI events
```java
// Create a new MidiProcessor:
MidiProcessor processor = new MidiProcessor(midi);

// Register for the events you're interested in:
EventPrinter ep = new EventPrinter("Individual Listener");
processor.registerEventListener(ep, Tempo.class);
processor.registerEventListener(ep, NoteOn.class);

// or listen for all events:
EventPrinter ep2 = new EventPrinter("Listener For All");
processor.registerEventListener(ep2, MidiEvent.class);

// Start the processor:
processor.start();
```
```java
// This class will print any event it receives to the console
public class EventPrinter implements MidiEventListener
{
    private String mLabel;

    public EventPrinter(String label)
    {
        mLabel = label;
    }

    @Override
    public void onStart(boolean fromBeginning)
    {
        if(fromBeginning)
        {
            System.out.println(mLabel + " Started!");
        }
        else
        {
            System.out.println(mLabel + " resumed");
        }
    }

    @Override
    public void onEvent(MidiEvent event, long ms)
    {
        System.out.println(mLabel + " received event: " + event);
    }

    @Override
    public void onStop(boolean finished)
    {
        if(finished)
        {
            System.out.println(mLabel + " Finished!");
        }
        else
        {
            System.out.println(mLabel + " paused");
        }
    }
}
```



### License
[Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0)
