Testing Click Applications
==========================

.. currentmodule:: click.testing

For basic testing, Click provides the :mod:`click.testing` module which
provides test functionality that helps you invoke command line
applications and check their behavior.

These tools should really only be used for testing as they change
the entire interpreter state for simplicity and are not in any way
thread-safe!

Basic Testing
-------------

The basic functionality for testing Click applications is the
:class:`CliRunner` which can invoke commands as command line scripts.  The
:meth:`CliRunner.invoke` method runs the command line script in isolation
and captures the output as both bytes and binary data.

The return value is a :class:`Result` object, which has the captured output
data, exit code, and optional exception attached.

Example::

    # example.py
    import click

    @click.command()
    @click.argument('name')
    def hello(name):
        click.echo('Hello %s!' % name)

Associated test(using pytest)::

    # test_hello.py
    from click.testing import CliRunner

    def test_hello_world():
        import example

        runner = CliRunner()
        result = runner.invoke(example.hello, ['Peter'])
        assert result.exit_code == 0
        assert result.output == 'Hello Peter!\n'

For subcommand testing, a subcommand name must be specified in the `args` parameter of :meth:`CliRunner.invoke` method.

Example::

    # example.py
    import click
    
    @click.group()
    @click.option('--debug/--no-debug', default=False)
    def cli(debug):
        click.echo('Debug mode is %s' % ('on' if debug else 'off'))

    @cli.command()
    def sync():
        click.echo('Syncing')

Associated test(using pytest)::

    # test_cli.py
    from click.testing import CliRunner

    def test_sync():
        import example

        runner = CliRunner()
        result = runner.invoke(example.cli, ['--debug', 'sync'])
        assert result.exit_code == 0
        assert 'Debug mode is on' in result.output
        assert 'Syncing' in result.output

Additional keyword arguments passed to ``.invoke()`` will be used to construct the initial Context object. For example, if you want to run your tests against a fixed terminal width you can use the following::

    runner = CliRunner()
    result = runner.invoke(example.cli, ['--debug', 'sync'], terminal_width=60)

File System Isolation
---------------------

For basic command line tools that want to operate with the file system, the
:meth:`CliRunner.isolated_filesystem` method comes in useful which sets up
an empty folder and changes the current working directory to.

Example::

    # example.py
    import click

    @click.command()
    @click.argument('f', type=click.File())
    def cat(f):
        click.echo(f.read())

Associated test (using pytest)::

    # test_cat.py
    from click.testing import CliRunner

    def test_cat():
        import example

        runner = CliRunner()
        with runner.isolated_filesystem():
            with open('hello.txt', 'w') as f:
                f.write('Hello World!')

            result = runner.invoke(example.cat, ['hello.txt'])
            assert result.exit_code == 0
            assert result.output == 'Hello World!\n'


Input Streams
-------------

The test wrapper can also be used to provide input data for the input
stream (stdin).  This is very useful for testing prompts, for instance::

    # example.py
    import click

    @click.command()
    @click.option('--foo', prompt=True)
    def bar(foo):
        click.echo('foo=%s' % foo)

Associated test (using pytest)::

    # test_bar.py
    from click.testing import CliRunner

    def test_prompts():
        import example

        runner = CliRunner()
        result = runner.invoke(example.bar, input='wau wau\n')
        assert not result.exception
        assert result.output == 'Foo: wau wau\nfoo=wau wau\n'

Note that prompts will be emulated so that they write the input data to
the output stream as well.  If hidden input is expected then this
obviously does not happen.
