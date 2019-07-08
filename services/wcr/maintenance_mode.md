# Maintenance mode

> This was implemented in the [Waste carriers frontend](https://github.com/DEFRA/waste-carriers-frontend) prior to the switch to AWS and the introduction of the front and back office apps in September 2018. It has never been used and actually just needs to be stripped out. However this guide is included here for reference until such time as that is done.

You can place the WCR service into a maintenance mode, especially useful if you have a backend task that needs to be done, which will impact the service but not require it to be taken down completely.

We use the [Turnout](https://github.com/biola/turnout) gem to do this. After simply running a rake task we can return a maintenance page to all requests made to the service. The maintenance page can include a message set at the time we turn it on, plus we can specify permitted routes i.e. these should behave as normal and the rest be blocked, and even whether only certain IP addresses can access the site.

For further details please checkout the [Turnout](https://github.com/biola/turnout) site, the following details cover a basic example of putting the WCR service into maintenance mode.

## Maintenance page generation

One limitation of the gem is that it cannot use a `.erb` file for the ```maintenance.html``` page. In testing this became an issue because all our assets are pre-compiled and then given a hash. Referring to the assets without the hash would result in them not being found and the page not being styled.

We have resolved this by creating our own [rake](https://github.com/ruby/rake) task for generating a static maintenance.html based on a `.html.erb` template. We have then incorporated this into a default maintenance rake task.

N.B. We assume you are running this in *production* hence the inclusion of environment args to the commands. If omitted it will default to *development*.

## Turn it on

On the server running the project call the following command from the terminal.

```bash
rake maintenance:default RAILS_ENV=production
```

This will automatically generate a `maintenance.html` file based on `maintenance_template.html.erb`, then will call `maintenance:start` passing in `/assets/*` as an allowed path and setting a default reason.

### Customised default task

You can still use the arguments the Turnout gem expects with the `default` task command. For example;

```bash
rake maintenance:default reason="The service will return at approximately 2pm today" allowed_paths="/registrations/search" RAILS_ENV=production
```

This would replace the default reason with the one specified, and add `/registrations/search` along with `/assets/*` to the allowed paths.

## Turn it off

When you are ready to turn off maintenance mode run the following command.

```bash
rake maintenance:stop
```

This will delete both the `maintenance.html` and `maintenance.yml` files generated by the `default` rake task. Its the presence of the `maintenance.yml` file that determines whether the site displays its maintenance page or not.

## Allowed IP's

The Turnout gem has the ability to permit requests from certain IP's to see the site as normal by passing in `allowed_ips="4.8.15.16"` as an argument. At this time though we have been unable to get this working in our production environment hence we do not cover it in the examples above.