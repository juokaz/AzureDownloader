# Azure Downloader

This is a Windows service which you can run inside your Web Role to sync website contents with a file stored in Blob storage. It holds some properties:

+ Runs inside a Web Role, no need for additional worker role
+ File stored in blob storage is a Zip archive, so downloads and uploads are very fast
+ Shared access url is used, rather than it connecting to Blob storage with admin rights, role itself can't change the blob or any of the files stored in a container
+ File is checked with HEAD request and only downloaded if new ETag is present, thus reduces bandwith out of an instance
+ Configurable refresh interval

This is somewhat close to what was used for [Azure+](http://cloud.webspecies.co.uk/).

## Configuration

This service only needs two configuration parameters: `APP_URL` and `APP_INTERVAL`. So first add these to `ServiceDefinition.csdef`:

	<ConfigurationSettings>
		<Setting name="APP_URL" />
		<Setting name="APP_INTERVAL" />
	</ConfigurationSettings>

And then configure as you want in `ServiceConfiguration.cscfg`.

## Auto Install

Installing this service is easy through a cmd script which you can run as a startup task (add this `<Task commandLine="install-service.cmd" executionContext="elevated" taskType="background" />` to Startup node), just create a `install-service.cmd` file with contents like:

	@echo off

	ECHO Starting APP update setup >> log.txt

	IF EXIST %WINDIR%\Microsoft.NET\Framework64 (
	%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /u /LogToConsole=false ..\Assets\AzureDownloader.exe >> log.txt 2>>err.txt
	%WINDIR%\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /i /LogToConsole=false ..\Assets\AzureDownloader.exe >> log.txt 2>>err.txt
	) ELSE (
	%WINDIR%\Microsoft.NET\Framework\v4.0.30319\installutil.exe /u /LogToConsole=false ..\Assets\AzureDownloader.exe >> log.txt 2>>err.txt
	%WINDIR%\Microsoft.NET\Framework\v4.0.30319\installutil.exe /i /LogToConsole=false ..\Assets\AzureDownloader.exe >> log.txt 2>>err.txt
	)

	net start "Azure Downloader" >> log.txt 2>>err.txt

	ECHO Completed APP update setup >> log.txt

	EXIT /B 0
