# Represent API: Images

[Represent](http://represent.opennorth.ca) is the open database of Canadian elected officials and electoral districts. It provides a [REST API](http://represent.opennorth.ca/api/) to boundary, representative, and postcode resources.

This repository stores scripts to extract images from the database. The [represent-canada](http://github.com/opennorth/represent-canada) repository is what's running at [represent.opennorth.ca](http://represent.opennorth.ca/).

## Usage

Download and resize images:

    bundle
    rake download
    rake resize

Consult the image sizes and aspect ratios:

    rake sizes
    rake ratios

Create an HTML page with all the 60x90 images:

    rake html
    open images.html

Remove all images:

    rake clean

## Bugs? Questions?

This project's main repository is on GitHub: [http://github.com/opennorth/represent-canada-photos](http://github.com/opennorth/represent-canada-photos), where your contributions, forks, bug reports, feature requests, and feedback are greatly welcomed.

Copyright (c) 2014 Open North Inc., released under the MIT license
