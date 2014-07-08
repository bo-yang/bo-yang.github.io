---
layout: post
title: Find Top Source IP Addresses in Distributed Systems
categories: 
- Distributed Systems
- Perl
- Network
tags:
- Distributed Systems
- Perl
- Network
status: publish
type: post
published: true
author: Bo Yang
---

The problem is:

> Determines the top 10 most common source IP addresses, and their hit rates, for a fleet of 1000 web servers within the last hour.
> The following assumptions may be used...
> * web servers are locally writing access logs in the [Apache Combined Log Format](http://httpd.apache.org/docs/current/logs.html#combined).
> * web servers are accessible by `ssh`.

My solution is:

1. For each server in the fleet, copy the `/var/log/httpd/access_log` to local directory by `ssh`.
2. Given a access_log, for each line in the log, extract the IP address, time/time zone and status code. 
3. For lines written in the past hour, count the occurrences of each IP address(number of requests) and numbers of sucessful operations(status code 1xx,2xx and 3xx in my implementation).
4. After counting logs of all servers, sort the IP addresses by their occurrences.
5. For the top 10 IP addresses, calculate their hit rates using equation:
	`hit rate = (# of successful operations) / (# of all requests)`

Assumptions are:

1. The names of servers in the fleet are stored in a file, such as 'fleet_servers'. The servers can be geographically distributed in different cities/countries.
2. This script will be executed in a directory that can be read & written by the user. 
3. The report of the top 10 most common IPs and hit rates will be stored in current directory.
4. The SSH keys to remote servers have been generated and synced, so that there is no need to manually enter the passwords. 
3. Following Perl modules should be installed:
	* DateTime - used to determine if the logs are older than 1 hour.
	* Parallel::ForkManager - check multiple logs in concurrently.

Following is my Perl implementation:

	use strict;
	use warnings;
	use POSIX qw(strftime);
	use DateTime;
	use Parallel::ForkManager;
	use Getopt::Long;
	use FindBin qw($Bin);
	use Cwd 'abs_path';
	
	my $ldt=DateTime->now(); # Get UTC date time
	my $input='fleet_servers'; # default file to store server list
	my $output="report_".strftime("%Y%d%m_%H%M%S",localtime()); # default file t ostore report
	my $login=`whoami`;
	my $access_log='/var/log/httpd/access_log'; # the default location of the access log
	my $time_range=1; # 1 hour by default
	my $num_ips=10;	# number of the most common source IP addresses
	my $num_process=30; # Max number of processes for parallel processing
	
	GetOptions(
		'input|i=s'  => \$input,
		'output|o=s' => \$output,
		'login|l=s'  => \$login,
		'alog|a=s'   => \$access_log,
		'trange|t=i' => \$time_range,
		'num_ip|n=i' => \$num_ips,
		'process|p=i'=> \$num_process,
		'help|?'	 => \&ShowUsage
	   ) or ShowUsage();
	
	
	die("ERROR: no input reads specied!\n") if (!$input);
	
	# Handle signals
	my $CleanupDone   = 0;
	$SIG{'ABRT'} = 'doCleanup';
	$SIG{'HUP'}  = 'doCleanup';
	$SIG{'INT'}  = 'doCleanup';
	$SIG{'QUIT'} = 'doCleanup';
	
	open IN, "<$input" or die("ERROR: cannot open file $input!");
	
	# Iteratively access each server. Assume that each line of the input file is 
	# a server name or IP address.
	my $clients={};
	my $pm = new Parallel::ForkManager($num_process);
	while (my $server = <IN>) {
		$pm->start and next; # do the fork
	
		chomp $server;
		my $tmplog=".${server}.access_log.tmp";
		`ssh ${login}@${server} "cat ${access_log}" > $tmplog`;
	
		open LOG, "<$tmplog" or die("ERROR: cannot open file $tmplog!");
		# Parse the access_log of this server
		while (my $line = <LOG>) {
			if($line =~ /^(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})/) {
				my $ip=$1;
				my ($rdt, $stat_code)=($line =~ m/\[(.*)\].*" (\d{3})/);
		
				# Calculate time differece between local and remote servers(in seconds)
				my $time_diff=CompareTime($ldt,$rdt);
				# Skip messages older than specified time range
				next if($time_diff > $time_range*3600);
		
				# Count requests and successful oprations
				if(not $clients->{$ip}){
					$clients->{$ip}->{'request'}=1;
					$clients->{$ip}->{'success'}=0;
				} else {
					$clients->{$ip}->{'request'}++;
				}
				# Only status code that begins with 1, 2 or 3 will be counted
				$clients->{$ip}->{'success'}++ if($stat_code =~ /[1-3]\d{2}/);
			} else {
				warn "No IP address found in ${tmplog}.";
			}
		}
		close LOG;
		unlink $tmplog; # remove temporary files
	
		$pm->finish;
	}
	$pm->wait_all_children; # wait for forked-processes
	
	close IN;
	
	# Find the most common IP addresses and calculate their hit rates
	open OUT, ">$output" or die("ERROR: cannot write file $output!");
	print OUT "IP Address \t Hit Rate \t # Req \t # Succ\n";
	print STDOUT "IP Address \t Hit Rate \t # Req \t # Succ\n";
	my $cnt=0;
	# Sort source IP addresses in descending order of requests
	foreach my $ip (sort {$clients->{$b}->{'request'} <=> $clients->{$a}->{'request'}} keys %{$clients}) {
		# Calculate hit rate
		$clients->{$ip}->{'hit rate'}=$clients->{$ip}->{'success'}/$clients->{$ip}->{'request'};
		my $hit_rate=sprintf("%.4f",$clients->{$ip}->{'hit rate'});
		print OUT "$ip \t $hit_rate \t $clients->{$ip}->{'request'} \t $clients->{$ip}->{'success'}\n";
		print STDOUT "$ip \t $hit_rate \t $clients->{$ip}->{'request'} \t $clients->{$ip}->{'success'}\n";
	
		$cnt++;
		last if($cnt>=$num_ips); # break out
	}
	
	close OUT;
	
	
	#
	# Compare two time and return the differences in seconds.
	# 
	# Inputs: 
	# 	local time(array), remote time(string)
	# Output: 
	# 	time difference(in seconds)
	#
	sub CompareTime {
		my $ldt=$_[0]; # local time, array
		my $remote_dt=$_[1]; # remote time, string
		
		my %month_map = (
			'Jan' => 1, 'Feb' => 2, 'Mar' => 3,
			'Apr' => 4, 'May' => 5, 'Jun' => 6,
			'Jul' => 7, 'Aug' => 8, 'Sep' => 9,
			'Oct' => 10,'Sep' => 11,'Dec' => 12
		);
	
		my ($rday,$rmonth,$ryear,$rhour,$rmin,$rsec,$rtz)=($remote_dt =~ m/(\d+)\/(\S+)\/(\d+):(\d+):(\d+):(\d+) ([+-]\d+)/);
		my $rdt = DateTime->new(
			year	=> $ryear,
			month	=> $month_map{$rmonth},
			day		=> $rday,
			hour	=> $rhour,
			minute	=> $rmin,
			second	=> $rsec,
			time_zone => $ldt->time_zone,
		);
	
		# Convert the remote time into UTC time zone.
		my $ltz='+0000';
		if($ltz ne $rtz) {
			my $abs_rtz=$rtz;
			if($abs_rtz =~ m/[+-].*/) {
				$abs_rtz=substr $abs_rtz, 1; # remove the leading +/-
			}
			# The ISO 8601 offset from UTC in timezone linke +0700 (1 minute=1, 1 hour=100)
			my $tz_diff=$abs_rtz/100*3600; # time zone diff in seconds
			my $dur = DateTime::Duration->new(
				years       => 0,
		      	months      => 0,
		      	weeks       => 0,
		      	days        => 0,
		      	hours       => 0,
		      	minutes     => 0,
		      	seconds     => $tz_diff,
		      	nanoseconds => 0
			);
			# Add/subtract time zone influence
			if($rtz =~ /\-\d+/) {
				$rdt->add_duration($dur);
			} else {
				$rdt->subtract_duration($dur);
			}
		}
		
		# Calculate the absolute time differences in seconds
		my $time_diff=$ldt->subtract_datetime_absolute($rdt);
		return $time_diff->in_units('seconds');
	}
	
	#
	# Show usgage
	# 
	sub ShowUsage {
		my $proc=`basename $0`;
		chomp($proc);
	
		print STDOUT "Usage: $proc [-i server_list] [-o report] [-l login] [-a access_log] [-t time_range] [-n ip_num] [-p process_num]\n";
		print STDOUT "where:\n";
		print STDOUT "  server_list\t - list of servers in the pool, optional\n";
		print STDOUT "  report\t - report file, optional \n";
		print STDOUT "  login\t\t - login to admin servers, optional\n";
		print STDOUT "  access_log\t - path of Apache access_log, optional\n";
		print STDOUT "  time_range\t - range of time to be analysed(unit: hour), optional\n";
		print STDOUT "  ip_num\t - number of the top source IPs, optional\n";
		print STDOUT "  process_num\t - number of concurrent processes, optional\n";
		
		exit;
	}
	
	#
	# Clean up before exit
	#
	sub doCleanup {
		if($CleanupDone == 1) {
			exit;
		}
	
		`rm .*.access_log.tmp`; # remove temp files
		$CleanupDone = 1;
		exit 0;
	} ## end sub doCleanup
