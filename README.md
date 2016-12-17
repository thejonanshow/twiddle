# Twiddle

Twiddle is a gem to process MIDI input from Palette hardware interfaces.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'twiddle'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install twiddle

## Usage

The Palette software is required to use this gem: https://palettegear.com/start

Download and install Palette then setup a MIDI profile:

![new_midi_screenshot](https://cloud.githubusercontent.com/assets/270746/21290147/0a92f156-c464-11e6-9c2d-c542236c95b3.png)

Then setup your individual devices:

![button_setup_screenshot](https://cloud.githubusercontent.com/assets/270746/21290209/757f5426-c466-11e6-9168-a63bc16bf8c2.png)

Take note of the options like "note" and "octave", Twiddle will need them later:

![note_octave_screenshot](https://cloud.githubusercontent.com/assets/270746/21290214/99cb0776-c466-11e6-98d9-f0a12c9768ee.jpg)

```
require "twiddle"
```

Then setup and name the interfaces you want to use. Twiddle currently supports
buttons, sliders and dials.

```
Twiddle.configure do |twiddle|
  # Choose a name and include the options from the MIDI profile you setup.
  # twiddle.button.<name>   = { <options_from_palette> }

  twiddle.button.launch     = { note: "D#", octave: -2 }
  twiddle.button.reticulate = { note: "F#", octave: -1 }
  twiddle.slider.warp       = { cc: 1 }
  twiddle.dial.transporter  = { note: "C#", octave: 0, cc: 4 }
end
```

Now you can write define your classes to use the input however you want:

```
class Launch < Twiddle::Button
  # class names are important; the Launch class runs code for the configuration
  # "twiddle.button.launch".

  def press
    # This code will run whenever the "launch" button is pressed.
  end
end
```

Define a class for a slider:

```
class Reticulate < Twiddle::Slider
  def slide(value)
    # This method is called whenever the reticulate slider is moved.
    # The "value" will be the new value for the slider position. Note that when
    # someone is actively sliding this value will change a lot. If you only care
    # about the value they settle on use the #slide_final method instead.
  end

  def slide_final(value)
    # This method will be called when someone settles on a value for a
    # configurable timeout (the default is half a second).
  end
end
```

Define a class for a dial:

```
class Transporter < Twiddle::Dial
  def press(value)
    # Because dials can have a value when they're pressed we pass that to #press
  end

  def rotate(value)
    # Called whenever value changes more than the dial.ignore setting  so we
    # automatically disregard accidental dial bumps. If you want fine-grained
    # control over your dials may consider setting this to 0.
  end

  def rotate_final(value)
    # Called when the dial settles on a value for longer than half a second.
  end
end
```

You can set options for each of the devices individually:

```
# Configure this particular slider to wait 2 seconds after the slider stops
# moving before calling #slide_final:
Twiddle.configure do |twiddle|
  twiddle.slider.reticulate.timeout = 2
end
```

You can also set default options for all devices of a particular type:
```
# Configure ALL sliders to timeout after 1 second:
# This setting will be ignored by any sliders that have their own specific
# timeout configured.
Twiddle.configure do |twiddle|
  twiddle.slider.timeout = 2
end
```

Note that you can configure global and individual settings together:
```
Twiddle.configure do |twiddle|
  twiddle.slider.timeout = 2
  twiddle.dial.timeout = 1
  twiddle.dial.ignore = 3

  twiddle.dial.transporter.timeout = 3
  twiddle.dial.transporter.ignore = 0
end
```

To see all of the configuration settings with their values:
```
Twiddle.config
# =>
# {
#   slider: {
#     timeout: 0.5, # default,
#     global: {
#       timeout: 1 # override default timeout to be 1
#     },
#     custom: {
#       reticulate: {
#         timeout: 2 # reticulate timeout
#       }
#     }
#   },
#   dial: {
#     ignore: 1, # default,
#     timeout: 0.5 # default,
#     global: {},
#     custom: {
#       transporter: {
#         ignore: 0
#       }
#     }
#   }
# }
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/thejonanshow/twiddle. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

