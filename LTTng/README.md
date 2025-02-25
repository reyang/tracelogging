# TraceLogging for LTTNG

The TraceLogging for LTTNG project enables structured event emission through LTTNG via the same set of macros that are
supported by the publicly available [TraceLogging for ETW](https://docs.microsoft.com/windows/win32/tracelogging/trace-logging-about)
in the Windows SDK.

## Examples

### Definition, registration, unregistration

To emit events through a TraceLogging provider, you must, at minimum:

1) Define a provider handle with a **unique provider UUID**.
2) At runtime, before writing any events, call TraceLoggingRegister.
3) Write events using TraceLoggingWrite.
4) At shutdown, call TraceLoggingUnregister.

For example:

```cpp
// Provider UUID: 22731e9c-e31a-4484-98d6-efa23d791456
TRACELOGGING_DEFINE_PROVIDER(
    WidgetProvider,
    // By convention: "Company.Organization.Team.Component"
    "Microsoft.Windows.Fundamentals.Widget",
    (0x7ea733d9, 0xf4eb, 0x4593, 0x8a, 0x8a, 0x8f, 0x9e, 0x75, 0xb0, 0x80, 0x04));

TraceLoggingRegister(WidgetProvider);

int x = 5;
TraceLoggingWrite(WidgetProvider,
    "TestEvent", // Event name
    TraceLoggingValue(x, "xValue"));

TraceLoggingUnregister(WidgetProvider);
```

### Logging LTTNG events through TraceLogging API

In C++, TraceLoggingValue will attempt to deduce the logging type for you. There are specific per-type macro variants for C or for cases
where you wish to explicitly pick the logged type yourself. The TraceLogging header contains extensive comments on the type system.
The header also contains more detailed descriptions of keywords and levels, which are ways to split events into logical groupings.
Below is an example of logging a more complex event:

```cpp
TraceLoggingWrite(
    WidgetProvider,
    "FinishUpload",
    TraceLoggingKeyword(KeywordEnum::Uploader),
    TraceLoggingLevel(WINEVENT_LEVEL_INFO),
    TraceLoggingValue(uploadUrl.c_str(), "Url"),
    TraceLoggingValue(statusCode, "HttpStatus"),
    TraceLoggingValue(headers.c_str(), "ResponseHeaders"));
```

### Consuming LTTNG events logged through TraceLogging API

To consume TraceLogging events sent through LTTNG, you will need the lttng-tools package (see [Dependencies](#Dependencies)). You can collect trace points using event wildcards:

```bash
lttng create
lttng enable-event -u Microsoft.Windows.Fundamentals.TestProvider:*
lttng start

read -p "Run your program here..."

lttng stop
lttng view
```

For more information, see the [LTTNG Documentation](https://lttng.org/docs/v2.10/).

## Dependencies

This project depends on the lttng-ust library. To build this library, you will need liblttng-ust-dev version 2.10 or later.
The library will compile with 2.7 or later, but some things might not work perfectly. The library has been tested up through
version 2.13.

```bash
sudo apt update
sudo apt install liblttng-ust-dev
```

If your normal package repository uses an older version of LTTNG, consider using the ppa:lttng/stable-2.10 repository to get LTTNG 2.10:

```bash
sudo apt-add-repository ppa:lttng/stable-2.10 -y
sudo apt -y update
sudo apt install liblttng-ust-dev
```

To listen to TraceLogging events, you will need lttng-tools. Note that the tools are required for event collection
but are not required for your program to run.

```bash
sudo apt install lttng-tools
```

## Integration

TraceLogging builds as an interface library with a dependency on a static library and requires C++11 and C99. The project creates a CMake compatible 'tracelogging' target which you can use for linking against the library.

1. Add this project as a subdirectory in your project, either as a git submodule or copying the code directly.
2. Add that directory to your top-level CMakeLists.txt with 'add_subdirectory'. This will make the target 'tracelogging' available.  
3. Add the 'tracelogging' target to the target_link_libraries of any target that will use TraceLogging.

## Reporting Security Issues

Security issues and bugs should be reported privately, via email, to the
Microsoft Security Response Center (MSRC) at <[secure@microsoft.com](mailto:secure@microsoft.com)>.
You should receive a response within 24 hours. If for some reason you do not, please follow up via
email to ensure we received your original message. Further information, including the
[MSRC PGP](https://technet.microsoft.com/en-us/security/dn606155) key, can be found in the
[Security TechCenter](https://technet.microsoft.com/en-us/security/default).

## Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).

For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Contributing

Want to contribute? The team encourages community feedback and contributions. Please follow our [contributing guidelines](../CONTRIBUTING.md).

We also welcome [issues submitted on GitHub](https://github.com/Microsoft/TraceLogging/issues).

## Project Status

This project is currently in active development.

## Contact

The easiest way to contact us is via the [Issues](https://github.com/microsoft/TraceLogging/issues) page.
