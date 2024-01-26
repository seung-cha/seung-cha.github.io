---
layout: post
title:  "Implementation of Input Buffer for Fighting Games"
date:   2024-01-26 11:50:17 +1100
categories: Coding
---
Fighting games are (in)famous for their unique input system where the player must sequencially input a set of keys within a short window to perform a special attack. Such a system is called motion input.

Despite the inherent difficulty in successfully performing a motion input which leads to a lack of popularity of the genre, the motion input system has always been the foundation of traditional fighting games. 

Motion input system works roughly by storing inputs in each frame and checking if the player performed a motion input. Different fighting games implement the system differently hence each game feels different to play. For example, in Street Fighter 6, the motion input to perform a dragon punch can be done without pressing the forward button at all. Such a difference also allows users to learn the system and strategises their gameplay.

In this article, I will be implementing a simple motion input system that is easy to expand in Unity.
I will assume you are familiar with the numpad notation.

Context
---
The system will receive 4 directional keys (up, down, left, right) and 3 attack keys (light, medium, heavy) of a typical 2D fighting game. Although not every fighting game follows this structure and yours will also likely differ too, implementation should remain the same or similar regardless (unless your game is 4D). It will run in 60 fps.

Input Buffer
---
Input buffer is data structure behind the scene. It is a contiguous array of fixed size storing input in each frame. I chose to work with `List` instead of `LinkedList` because:
* random access is O(1).
* The interface is much more convenient to work with
* random access happens more often than insertion despite insertion being called every frame.

It is absolutely fine to use LinkedList to implement input buffer. Validating motion input would require a bit more of work, though.

`Stack` will not work because we need to access elements in the middle of the list without popping them. Suppose you want to check if the user can perform 236. How would you check the 2 and 3 inputs without popping elements in Stack?

Implementation of Input Buffer is roughly as follows:
{% highlight cs %}
public class InputBuffer
{
    /// <summary>
    /// How long the buffer should store input?Confirming
    /// </summary>
    public const int BUFFERLEN = 60;
    private List<InputButton[]> buffer;
}

{% endhighlight %}
Here, `InputButton` is a class storing input data. The buffer stores an array of `InputButton` because there are n buttons.

The `InputButton` class looks like this:
{% highlight cs %}
public class InputButton
{
    public const Button NEUTRAL = 0;
    public const Button UP = 1;
    public const Button DOWN = 2;
    public const Button FORWARD = 3;
    public const Button BACK = 4;
    public const Button LIGHT = 5;
    public const Button MEDIUM = 6;
    public const Button HEAVY = 7;

    public const Button DOWNFORWARD = 8;
    public const Button DOWNBACK = 9;
    public const Button UPFORWARD = 10;
    public const Button UPBACK = 11;

    /// <summary>
    /// Number of buttons
    /// </summary>
    public const Button LEN = 8;

    public int Frame = 1;
    public bool Released = false;
    public bool Consumed = false;
    public bool Transitioned = false;   // True if Released changed from the previous frame.
}
{% endhighlight %}
Each constant variables (0 ~ 7) represents their respective index of the buffer. `Button` is a type alias of `Byte`.
Constant variables from 8 to 11 are used to fetch directional inputs from the buffer.

`Frame` is the length of consecutive input. For example, if you stand still for 2 frames, in the next frame `InputButton` for `NEUTRAL` will have `Frame = 3`.

`Released` is self-explanatory: true if the user is pressing this button and false otherwise.

`Consumed` is a flag to indicate that the input is 'consumed'. This is to ensure that no same input causes the same action twice.
This variable is not used in this article but for more complex system, consider using it.

`Transitioned` indicates that the `Released` flag changed from the last frame. 

When inserting a new input in the buffer, we need to set the flags:
{% highlight cs %}
/// <summary>
/// Add a button input to current frame.
/// Multiple calls with the same argument override the button with the last call.
/// </summary>
/// <param name="button">Button to add, obtained from the Input class</param>
/// <param name="released">Is the button to add released in this frame?</param>
public void AddButton(Button button, bool released)
{
    InputButton input = new InputButton();
    input.Released = released;

    // Check if the button is being held (or released)
    if (buffer.Count > 1)
    {
        InputButton prev = buffer[1][button];
        if (prev.Released == released)
        {
            // Same button is being held (or released). Increment the frame counter.
            input.Frame = prev.Frame + 1;
        }
        else
        {
            // Button held state changed
            input.Transitioned = true;
        }
    }

    buffer[0][button] = input;
}
{% endhighlight %}


In each frame we insert a new array of `InputButton` at the front of the buffer and check if the size exceeds the buffer length. If so, delete the last (oldest) input. We initialise the array with the buttons being released. This way, older inputs slowly decay. We gradually fill the array with new buttons being held as user presses keys.

{% highlight cs %}
/// <summary>
/// Call this to receive a fresh set of input.
/// All inputs are initialised as released.
    /// </summary>
    public void NewFrame()
    {
        buffer.Insert(0, new InputButton[InputButton.LEN]);

        // Add buttons, as released.
        AddButton(InputButton.NEUTRAL, true);
        AddButton(InputButton.UP, true);
        AddButton(InputButton.DOWN, true);
        AddButton(InputButton.FORWARD, true);
        AddButton(InputButton.BACK, true);

        AddButton(InputButton.LIGHT, true);
        AddButton(InputButton.MEDIUM, true);
        AddButton(InputButton.HEAVY, true);

        // Discard the oldest set of input if there is one.
        if (buffer.Count > BUFFERLEN)
        {
            buffer.RemoveAt(buffer.Count - 1);
        }
    }
{% endhighlight %}

And lastly, a function to clear the buffer.
{% highlight cs %}
/// <summary>
/// Wipe out the buffer.
/// </summary>
public void Clear()
{
    buffer.Clear();
}
{%endhighlight %}

That's all for input buffer. Receiving and storing input in the input buffer is not hard. What is tricky is how to use this.
Before we move on to implementing motion inputs, here are a few points we need to be aware of.


- Motion inputs have priorities  

Which one should come out if you press 6236. a DP? or a fireball? In most games 623 takes priority over 236. Similarly, 236236 over these two. 
- Motion inputs have to be checked every frame

Even if only directional keys are pressed, we still need to check if a motion input is done. Generally this is to check if a dash or a backdash is performed. In Tekken, some characters can perform their special manoeuvre with directional keys only (e.g wave dash)
- Input needs to be consumed

Suppose you are implementing a wave dash mechanics (6N23 to perform a special dash). If this is successfully performed, the set of inputs that contributes to wave dash should not be used again. Your character will perform another wave dash if it still exists in the buffer. Another solution to this is to decrease the maximum size of the input buffer. This way, the wave dash input decays naturally when your character is ready to perform another wave dash.

- It all depends on your design choice

I suppose there is no right way to implement input buffer and motion inputs - Every game is different. Some games may not even have complex motion inputs. How you use the data in the buffer is totally up to the designer.


One last thing, here are functions to access inputs in the input buffer:
{% highlight cs %}
/// <summary>
/// Find the non-diagonal button in the buffer with given conditions.
/// If button is directional, strictly find the button (i.e ignore the button if other directional buttons are pressed on the same frame)
/// </summary>
/// <param name="button">Button to find</param>
/// <param name="released">Whether the button should be released or not.</param>
/// <param name="frame">How long ago this button was received?</param>
/// <param name="strict">If true, find the button whose Transitioned field is true (released to pressed or pressed to released). This is always true if released is true.</param>
/// <param name="delay">If given, ignore buttons more recent than this.</param>
/// <returns>Object if found, null otherwise.</returns>
public InputButton FindButton(Button button, bool released,
                                out int frame, int delay = 0, bool strict = false)
{
    frame = delay;

    for (int i = delay; i < buffer.Count; i++)
    {
        InputButton inputButton = buffer[i][button];

        if ((inputButton.Transitioned && inputButton.Released == released) ||
            (!strict && !inputButton.Released))
        {
                if (!released && button <= 4)
                {
                    // See if other keys are pressed
                    bool other = false;

                    for (int j = 0; j <= 4; j++)
                    {
                        if (button == j)
                        {
                            continue;
                        }

                        if (!buffer[i][j].Released)
                        {
                            other = true;
                            break;
                        }
                    }

                if (other)  // Other key is pressed. Do not return this.
                {
                    continue;
                }

            }

            return inputButton;
        }

        frame++;
    }

    return null;
}
{% endhighlight %}
This function returns the earliest occurrence of input with given conditions.
Each parameter and condition is as follows:
* `released`: If true, look for the inputs being held.
* `delay`: Do not consider inputs received strictly earlier than this value. If `delay = 3`, ignore the first two inputs in the buffer.
* `strict`: If true, only look for the inputs on their first frame of being pressed or released. This field is always true when `released = false`.
* `frame`: is the frame on which the valid input is found. If `frame = 15`, it implies the valid input was received 15 frames ago.

Lastly, if the input-to-find is a directional input, no other directional keys should be pressed on the same frame.


A similar function is implemented for diagonal inputs:
{% highlight cs %}
/// <summary>
/// Strictly find the frame at which the provided diagonal button was pressed.
/// Returns the button with the minimum frame if exists, null otherwise.
/// </summary>
/// <param name="button"></param>
/// <param name="frame"></param>
/// <param name="delay"></param>
/// <returns></returns>
public InputButton FindDiagonalButton(Button button, out int frame, int delay = 0)
{
    Button btn1, btn2;
    frame = 0;

    switch (button)
    {
        case InputButton.DOWNFORWARD:
            btn1 = InputButton.DOWN;
            btn2 = InputButton.FORWARD;
            break;
        case InputButton.DOWNBACK:
            btn1 = InputButton.DOWN;
            btn2 = InputButton.BACK;
            break;
        case InputButton.UPFORWARD:
            btn1 = InputButton.UP;
            btn2 = InputButton.FORWARD;
            break;
        case InputButton.UPBACK:
            btn1 = InputButton.UP;
            btn2 = InputButton.BACK;
            break;

        default:
            return null;
    }

    int frame1, frame2;

    InputButton button1 = FindEarliestButton(btn1, false, out frame1, delay);
    InputButton button2 = FindEarliestButton(btn2, false, out frame2, delay);

    while (frame1 != frame2 && button1 != null && button2 != null)
    {
        if (frame1 > frame2)
        {
            button2 = FindEarliestButton(btn2, false, out frame2, frame1);
        }
        else
        {
            button1 = FindEarliestButton(btn1, false, out frame1, frame2);
        }
    }

    frame = frame1;

    if (button1 == null || button2 == null)
    {
        return null;
    }

    return button1.Frame > button2.Frame ? button2 : button1;
}


/// <summary>
/// Get the frame of the earliest occurrence of given button.
/// </summary>
/// <returns></returns>
private InputButton FindEarliestButton(Button button, bool released, out int frame, int delay = 0)
{
    frame = delay;
    for (int i = delay; i < buffer.Count; i++)
    {
        InputButton[] btn = buffer[i];

        if (btn[button].Released == released)
        {
            return btn[button];
        }

        frame++;
    }

    return null;
}
{% endhighlight %}

Now we have those interfaces ready, we can start implementing motion inputs.


Motion Inputs
---
As I mentioned earlier, motion inputs has priorities. 623 should be recognised prior to 236. The easiest solution I can think of is to implement motion inputs with the chain of responsibility design pattern.

Each motion input will be stored in a motion input chain. On every frame, the chain sequencially checks for different motion inputs.
This way, motion inputs can have priorities and changing order is also as simple as swapping lines.

{% highlight cs %}
public class CommandChain
{
    private List<ICommand> commands;

    public CommandChain()
    {
        commands = new List<ICommand>();
    }

    public void AddCommand(ICommand command)
    {
        commands.Add(command);
    }

    public void Update(InputBuffer buffer)
    {
        foreach(ICommand command in commands)
        {
            if(command.ProcessCommand(buffer))
            {
                break;
            }
        }
    }
}
{% endhighlight %}
If a motion input successfully comes out, we do not want to check for any other motion inputs on the same frame.

The interface class simply implements one function.
{% highlight cs %}
public interface ICommand
{
    bool ProcessCommand(InputBuffer buffer);
}
{% endhighlight %}

Checking for motion input is simply checking for sequencial inputs, working reverse.
For example, if we want to check if 236 is performed, we first would check if 6 is in the buffer. If so, check if 3 is in the buffer and then finally 2.

While doing so we want to make sure the inputs are performed within a window.
This part will largely differ depending on your own preferences. I will not talk much about motion input checking.
Here is my implementation of 236:

{% highlight cs %}
public class Command_Fireball : ICommand
{
    private int tol = 20;   // Input window

    public bool ProcessCommand(InputBuffer buffer)
    {
        // Confirm that 236 is pressed in given delay.
        int frame;
        InputButton btn = buffer.FindButton(InputButton.LIGHT, false, out frame, 0, true);

        if (btn == null || frame > tol)
        {
            // Button is not pressed or was pressed too late.
            return false;
        } 

        int prog = frame;
        InputButton fwd = buffer.FindButton(InputButton.FORWARD, false, out frame, prog);

        if (fwd == null || frame > tol)
        {
            return false;
        }

        prog += frame;

        // Get the down foward input.
        InputButton df = buffer.FindDiagonalButton(InputButton.DOWNFORWARD, out frame, prog);

        // If both are null, return false.
        if (df == null || frame > tol)
        {
            return false;
        }

        prog += frame;

        // lastly, get the down (or down back) button.
        InputButton dwn = buffer.FindButton(InputButton.DOWN, false, out frame, prog + 1);

        int frame2;
        InputButton db = buffer.FindDiagonalButton(InputButton.DOWNBACK, out frame2, prog + 1);

        if (((dwn == null) || frame > tol) && ((db == null) || frame2 > tol))
        {
            return false;
        }

        //Fireball command is casted.
        Debug.Log("Fireball is casted!");
        buffer.Clear();     // Wipe out the buffer

        return true;
    }

}
{% endhighlight %}

That's everything. Now you can start implementing more complex logic to polish your system and have your game unique.
You can check out the full implementation of input buffer and watch a demonstration video on my [repo][my-repo].

[my-repo]: https://github.com/seung-cha/fighting-game-motion-input/tree/main