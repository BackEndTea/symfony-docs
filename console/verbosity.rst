Verbosity Levels
================

Console commands have different verbosity levels, which determine the messages
displayed in their output. By default, commands display only the most useful
messages, but you can control their verbosity with the ``-q`` and ``-v`` options:

.. code-block:: terminal

    # suppress all output, including errors
    $ php bin/console some-command --silent

    # suppress all output (even the command result messages) but display errors
    $ php bin/console some-command -q
    $ php bin/console some-command --quiet

    # normal behavior, no option required (display only the useful messages)
    $ php bin/console some-command

    # increase verbosity of messages
    $ php bin/console some-command -v

    # display also the informative non essential messages
    $ php bin/console some-command -vv

    # display all messages (useful to debug errors)
    $ php bin/console some-command -vvv

.. versionadded:: 7.2

    The ``--silent`` option was introduced in Symfony 7.2.

The verbosity level can also be controlled globally for all commands with the
``SHELL_VERBOSITY`` environment variable (the ``-q`` and ``-v`` options still
have more precedence over the value of ``SHELL_VERBOSITY``):

=====================  =========================  ===========================================
Console option         ``SHELL_VERBOSITY`` value  Equivalent PHP constant
=====================  =========================  ===========================================
``--silent``           ``-2``                     ``OutputInterface::VERBOSITY_SILENT``
``-q`` or ``--quiet``  ``-1``                     ``OutputInterface::VERBOSITY_QUIET``
(none)                 ``0``                      ``OutputInterface::VERBOSITY_NORMAL``
``-v``                 ``1``                      ``OutputInterface::VERBOSITY_VERBOSE``
``-vv``                ``2``                      ``OutputInterface::VERBOSITY_VERY_VERBOSE``
``-vvv``               ``3``                      ``OutputInterface::VERBOSITY_DEBUG``
=====================  =========================  ===========================================

It is possible to print a message in a command for only a specific verbosity
level. For example::

    // ...
    use Symfony\Component\Console\Command\Command;
    use Symfony\Component\Console\Input\InputInterface;
    use Symfony\Component\Console\Output\OutputInterface;

    class CreateUserCommand extends Command
    {
        // ...

        public function execute(InputInterface $input, OutputInterface $output): int
        {
            $user = new User(...);

            $output->writeln([
                'Username: '.$input->getArgument('username'),
                'Password: '.$input->getArgument('password'),
            ]);

            // available methods: ->isSilent(), ->isQuiet(), ->isVerbose(), ->isVeryVerbose(), ->isDebug()
            if ($output->isVerbose()) {
                $output->writeln('User class: '.get_class($user));
            }

            // alternatively you can pass the verbosity level PHP constant to writeln()
            $output->writeln(
                'Will only be printed in verbose mode or higher',
                OutputInterface::VERBOSITY_VERBOSE
            );

            return Command::SUCCESS;
        }
    }

.. versionadded:: 7.2

    The ``isSilent()`` method was introduced in Symfony 7.2.

When the silent or quiet level are used, all output is suppressed as the default
:method:`Symfony\\Component\\Console\\Output\\Output::write` method returns
without actually printing.

.. tip::

    When using the ``silent`` verbosity, errors won't be displayed in the console
    but they will still be logged through the :doc:`Symfony logger </logging>` integration.

.. tip::

    The MonologBridge provides a :class:`Symfony\\Bridge\\Monolog\\Handler\\ConsoleHandler`
    class that allows you to display messages on the console. This is cleaner
    than wrapping your output calls in conditions. For an example use in
    the Symfony Framework, see :doc:`/logging/monolog_console`.

.. tip::

    The full exception stacktrace is printed if the ``VERBOSITY_VERBOSE``
    level or above is used.
