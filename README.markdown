Hack bundle for session beginning one click too early on login 
==================================

Purpose
---------------------

Because for a customer project we have a login form on every page and high trafic, we want to begin the session very late.
The goal is to get all the benefits of Varnish by caring very low on the cache policy at the VCL config level but more at the HTTP classical level.

The current issue points to this question : https://github.com/symfony/symfony/issues/3703

Assumption
---------------------
This assumption used for this temporary fix is really precise but allows only one 'login_check' route (for one firewall so)

To begin
---------------------

First, the auto_start session parameter must be disabled like this:

    # app/config/config.yml
    ...
    framework:
        ...
        session:
            auto_start:     false



Installation
---------------------

Add the bundle in the deps file of your project:

    # deps
    [HackSessionBundle]
        git=git@github.com:gillest/HackSessionBundle.git
        target=bundles/Gilles/Bundle/HackSessionBundle

Launch the bundle install command then:

    ./bin/vendors install

Add the bundle namespace to the autoload configuration file.

    # app/autoload.php
    <?php
    ...
    $loader->registerNamespaces(array(
        ...
        'Gilles'           => __DIR__.'/../vendor/bundles',
    ));
    
 Then, register the bundle in the kernel :
 
    # app/AppKernel.php
    <?php
    ...
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
                ...
                new Gilles\Bundle\HackSessionBundle\GillesHackSessionBundle(),
            );
            ...
        }
        ...
    }

At the configuration level, you need to provide the same parameter to both firewall and hack listener:

    # app/config/config.yml
    gilles_hack_session:
        login_check_path: %local_form_login_check_path%

Then, set to the same parameter the check_path you have set for the wanted firewall area:

    # app/config/security.yml
    security:
        ...
        firewalls:
            ...
            secured_area:
                pattern:    ^/demo/secured/
                form_login:
                    check_path: %local_form_login_check_path%

Now, you can set your URL that makes the login_check:

    # app/config/config.yml
    parameters:
        local_form_login_check_path: /demo/secured/login_check 

Result
---------------------     
The session does not need to exist now when the URL set to %local_form_login_check_path% in the firewall is hit.
