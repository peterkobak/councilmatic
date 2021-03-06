Councilmatic! [![Build Status](https://travis-ci.org/codeforamerica/councilmatic.png)](http://travis-ci.org/codeforamerica/councilmatic)
=============
City Council Legislative Subscription Service.

Contact Us
----------
- Join the mailing list at https://groups.google.com/group/councilmatic/
- Find us on irc.freenode.net in the #councilmatic room

Getting Started
---------------
**IMPORTANT:** If you are interested in setting up *Councilmatic* for your city, you'll want to start with a [sample local instance](https://github.com/mjumbewu/sample-local-councilmatic/). In most cases, this is the right place to start. If instead you are interested in making modification to the core of *Councilmatic*, proceed with these instructions.

Installation
------------
First check out the project code.

    $ git clone git://github.com/codeforamerica/councilmatic.git

To work on your own instance of Councilmatic, you should first get Python
installed. Follow the instructions for doing so on your platform. For example,
[Mac OS X Mountain Lion 10.8](http://hackercodex.com/guide/python-virtualenv-on-mac-osx-mountain-lion-10.8/).

We recommend setting up a virtual environment for working with any project, so that you can manage your project-specific dependencies. This is optional but recommended:

    virtualenv env --no-site-packages
    source env/bin/activate

Next, install the requirements for Councilmatic (we recommend working in a
virtual environment, but it's not strictly necessary).

    $ pip install -r requirements.txt


This file might not fully finish installing; if you get a hang-up on `pdfminer`, try [installing it manually](http://www.unixuser.org/~euske/python/pdfminer/#download) then re-run `pip install -r requirements.txt`.

Be sure to also install the scraper library for your city's legislative
management system. For cities using Granicus's latest version of Legistar,
this will be @fgregg's https://github.com/fgregg/legistar-scrape.

Non-Python requirements include:

* pdftotext and pdftohtml (use ``apt-get install poppler-utils`` on Ubuntu)


### Legislation source

Copy the file *local-councilmatic-sample/local_settings.py.template* to
*local-councilmatic-sample/local_settings.py*.  Fill in the `LEGISLATION` setting in this
file.  By default, it is set up to scrape from Philadelphia's legislation
system.

For example:

    LEGISLATION = {
        'SYSTEM': 'Granicus Legistar',
        'ROOT': 'http://lexington.legistar.com/',
        'ADDRESS_BOUNDS': [37.82714,-84.70459, 38.20797,-84.25964], # lat, lng, lat, lng
        'ADDRESS_SUFFIX': ', Lexington, KY',

        'SCRAPER': ('phillyleg.management.scraper_wrappers.sources.'
                    'hosted_legistar_scraper.HostedLegistarSiteWrapper'),
        'SCRAPER_OPTIONS': {
            'hostname': 'lexington.legistar.com',
            'fulltext': True,        # Load and store full text from PDFs?
            'sponsor_links': False,  # Are sponsor names in anchors/links?

            # Label overrides
            # ---------------
            'id_label': 'File #',
            'controlling_body_label': 'In control',
            'intro_date_label': 'File Created',
            'topics_label': 'Indexes',
        },
    }


### Database

Create a database for Councilmatic. Typically this is done like:

    createdb -T template_postgis councilmatic

where `template_postgis` is the name of your PostGIS database template. If you
do not yet have one, you can find instructions for getting your system ready for
Django and PostGIS online.  For example, here are instructions for
[Mac](https://gist.github.com/3188632), and
[Ubuntu](http://brandonkonkle.com/blog/2010/jul/19/setting-template_postgis-lucid/).
For other platforms, and for further instructions, the
[GeoDjango docs](https://docs.djangoproject.com/en/dev/ref/contrib/gis/install/#platform-specific-instructions)
are a good place to look.

**NOTE that PostGIS 2.0 is not compatible with Django 1.4.  As Councilmatic is
currently not set up to run on Django 1.5, you should install PostGIS 1.5**


Set up the project database and populate it with city council data (when the
syncdb command prompts you to create an administrative user, go ahead and do
so). There is a lot of data to be loaded, so downloading it all may take a
while.

    cd local-councilmatic-sample
    mkdir ../logs
    createuser -s -r postgres # Create the postgres role in PostgreSQL
    python manage.py syncdb # Create admin account when prompted.
    python manage.py migrate
    python manage.py loadlegfiles # For first-time installs.
    python manage.py updatelegfiles
    python manage.py rebuild_index # For searches. Say yes when prompted.
    python manage.py collectstatic # For js and css. Say yes when prompted.

For example, loading the database for the first time:

    (env)code/councilmatic/local-councilmatic-sample at owne-pc (master ✔)% python manage.py loadlegfiles
    No local copy of database exists.
    Downloading the database (~40M -- this may take a while)...

### Development server

Finally, to run the server:

    $ python manage.py runserver

Now, check that everything is working by browsing to http://localhost:8000/. Now
browse to http://localhost:8000/admin and enter the admin username and password
you supplied and you should have access to all of the legislative files!


Architecture
------------
The core of Councilmatic is implemented as a Django app. It cannot be run without a Django project. The local-councilmatic-sample folder contains a minimal Django project.


Copyright
---------
Copyright (c) 2010 Code for America Laboratories
See [LICENSE](https://github.com/cfalabs/open311/blob/master/LICENSE.mkd) for details.

[![Code for America Tracker](http://stats.codeforamerica.org/codeforamerica/philly_legislative.png)](http://stats.codeforamerica.org/projects/philly_legislative)
