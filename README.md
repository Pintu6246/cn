set ns [new Simulator]
set tf [open ex1.tr w]

$ns trace-all $tf

set nf [open ex1.nam w]
$ns namtrace-all $nf

set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]

$ns duplex-link $n0 $n2 2Mb 2ms DropTail
$ns duplex-link $n1 $n2 2Mb 2ms DropTail
$ns duplex-link $n2 $n3 0.4Mb 10ms DropTail

#$ns queue-limit $n0 $n1 5

set udp1 [new Agent/UDP]
$ns attach-agent $n0 $udp1
set null1 [new Agent/Null]
$ns attach-agent $n3 $null1
$ns connect $udp1 $null1

set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp1
$ns at 1.1 "$cbr1 start"

set tcp [new Agent/TCP]
$ns attach-agent $n3 $tcp
set sink [new Agent/TCPSink]
$ns attach-agent $n1 $sink
$ns connect $tcp $sink



set ftp [new Application/FTP]
$ftp attach-agent $tcp

$ns at 0.1 "$ftp start"
$ns at 10.0 "finish"

proc finish { } {
	global ns tf nf
	$ns flush-trace
	close $tf
	close $nf
	puts "Running nam...."
	exec nam ex1.nam &
	exit 0
      }
$ns run

BEGIN{

tcp_count=0;
udp_count=0;

if ($1 == "d" && $5 == "tcp") 
tcp_count++

if ($1 == "d" && $5 == "cbr") 
udp_count++;

}

END {
printf("Number of packet dropped in TCP %d\n", tcp_count);
printf("Number of packet dropped in UDP %d\n", udp_count);
}





set ns [new Simulator]
set tf [open prog2.tr]
$ns trace-all $tf
set nf [open prog2.nam w]
$ns namtrace-all $nf
set cwind [open win2.tr w]
$ns color 1 Blue

set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]
set n6 [$ns node]
$ns duplex-link $n1 $n3 2Mb 2ms DropTail
$ns duplex-link $n2 $n3 2Mb 2ms DropTail
$ns duplex-link $n3 $n4 2Mb 2ms DropTail
$ns duplex-link $n4 $n5 2Mb 2ms DropTail
$ns duplex-link $n4 $n6 2Mb 2ms DropTail

set tcp0 [new Agent/TCP]
$ns attach-agent $n1 $tcp0 
set sink0 [new Agent/TCPSink]
$ns attach-agent $n6 $sink0
$ns connect $tcp0 $sink0
set ftp [new Application/FTP]
$ftp attach-agent $tcp0

set tcp1 [new Agent/TCP]
$ns attach-agent $n2 $tcp1
set sink1 [new Agent/TCPSink]
$ns attach-agent $n5 $sink1
$ns connect $tcp1 $sink1
set tel1 [new Application/Telnet]
$tel1 attach-agent $tcp1
$ns at 0.1 "$tel1 start"
$ns at 10 "finish"

proc plotWindow {tcpSource file} {
    global ns 
    set time 0.01 
    set now [$ns now]
    set cwd0 [$tcpSource set cwnd_]
    puts $file "$now $cwd0"
    $ns at [expr $now+$time] "plotWindow $tcpSource $file"
}
proc finish { } {
        global ns tf nf cwind
        $ns flush-trace
        close $tf 
        close $nf 
        puts "Running nam..."
        exec nam prog2.nam &
        exec xgraph win2.tr &
        exit 0
}

BEGIN{

last=0
tcp_sz=0
cbr_sz=0
tot_sz=0
}
{
    action=$1;
    time=$2;
    from=$3;
    to=$4;
    type=$5;
    pktsize=$6;
    flow_id=$8;
    src=$9;
    dst=$10;
    seq_no=$11;
    packet_id=$12;
    if(type=="tcp" && action="r" && to=="3")
    tcp_sz+=pktsize
    if(type=="cbr" && action="r" && to =="3")
    cbr_sz+=pktsize
tot_sz+=pktsize
}
END {
print time, (tcp_sz*8/1000000)
print time, (tcp_sz*8/1000000),(tot_sz*8/10000000)
}




set ns [new Simulator]
set tf [open ex4.tr w]
$ns trace-all $tf
set nf [open ex4.nam w]
$ns namtrace-all $nf
set cwind [open win4.tr w]
$ns color 1 Blue
$ns color 2 Red

$ns rtproto DV

set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]

$ns duplex-link $n0 $n1 0.3Mb 10ms DropTail
$ns duplex-link $n0 $n2 0.3Mb 10ms DropTail
$ns duplex-link $n2 $n3 0.3Mb 10ms DropTail
$ns duplex-link $n1 $n4 0.3Mb 10ms DropTail
$ns duplex-link $n3 $n5 0.5Mb 10ms DropTail
$ns duplex-link $n4 $n5 0.5Mb 10ms DropTail

$ns duplex-link-op $n0 $n1 orient right-up
$ns duplex-link-op $n0 $n2 orient right-down
$ns duplex-link-op $n1 $n4 orient right
$ns duplex-link-op $n2 $n3 orient right
$ns duplex-link-op $n4 $n5 orient right-down
$ns duplex-link-op $n3 $n5 orient right-up

set tcp [new Agent/TCP]
$ns attach-agent $n0 $tcp 
set sink [new Agent/TCPSink]
$ns attach-agent $n5 $sink
$ns connect $tcp $sink
$tcp set fid_ 1
set ftp [new Application/FTP]
$ftp attach-agent $tcp

$ns rtmodel-at 1.0 down $n1 $n4
$ns rtmodel-at 3.0 up $n1 $n4
$ns at 0.1 "$ftp start"
$ns at 12.0 "finish"

proc plotWindow { tcpSource file } {
    global ns
    set time 0.01
    set now [$ns now]
    set cwnd [$tcpSource set cwnd_]
    puts $file "$now $cwnd"
    $ns at [expr $now+$time] "plotWindow $tcpSource $file"
}
$ns at 1.0 "plotWindow $tcp $cwind"
proc finish { } {
    global ns tf nf cwind
    $ns flush-trace
    close $tf 
    close $nf 
    exec nam ex4.nam &
    exec xgraph win4.tr &
    exit 0
}
$ns run



set ns [new Simulator]

set tf [open p4.tr w]
$ns trace-all $tf

set nf [open p4.nam w]
$ns namtrace-all $nf

set s [$ns node]
set c [$ns node]
$ns color 1 Blue

$s label "Server"
$c label "Client"

$ns duplex-link $s $c 10Mb 10ms DropTail
$ns duplex-link-op $s $c orient right

set tcp0 [new Agent/TCP]
$ns attach-agent $s $tcp0

$tcp0 set packetSize_ 1500

set sink0 [new Agent/TCP]
$ns attach-agent $c $sink0

$ns connect $tcp0 $sink0

set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0

$tcp0 set fid_ 1

proc finish {} {
	global ns tf nf
	$ns flush-trace
	close $tf
	close $nf
	exec nam p4.nam &
	exec awk -f p4transfer.awk p4.tr &
	exec awk -f p4convert.awk p4.tr > convert.tr &
	exec xgraph convert.tr -geometry 800*400 -t "bytes_received_at_client" -x "time_in_secs" -y "bytes_in_bps" &
}

$ns at 0.01 "$ftp0 start"
$ns at 15.0 "$ftp0 stop"
$ns at 15.1 "finish"
$ns run


# AWK script to calulate the time required to transfer the 10 MB file from the server to client
BEGIN {
	count=0;
	time=0;
    total_bytes_sent =0;
    total_bytes_received=0;
        
}
{
	if ( $1 == "r" &&  $4 == 1 && $5 == "tcp")
		total_bytes_received += $6;

    if($1 == "+" &&  $3 == 0 && $5 == "tcp")
		total_bytes_sent +=  $6;
} 
END {
       system("clear");
       printf("\n Transmission time required to transfer the file is %f",$2);
       printf("\n Actual data sent from the server is %f Mbps",(total_bytes_sent)/1000000);
       printf("\n Data Received by the client is %f Mbps\n",(total_bytes_received)/1000000);            	 
} 


BEGIN{
	count=0;
	time=0;
}
{
	if($1=="r" && $4==1 && $5=="tcp")
	{
	count+=$6;
	time=$2;
	printf("\n%f\t%f",time,(count)/1000000);
}
}
END{
}


set ns [new Simulator -multicast on]


set tf [open mcast.tr w]
$ns trace-all $tf


set fd [open mcast.nam w]
$ns namtrace-all $fd
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]
set n6 [$ns node]
set n7 [$ns node]


$ns duplex-link $n0 $n2 1.5Mb 10ms DropTail
$ns duplex-link $n1 $n2 1.5Mb 10ms DropTail
$ns duplex-link $n2 $n3 1.5Mb 10ms DropTail
$ns duplex-link $n3 $n4 1.5Mb 10ms DropTail
$ns duplex-link $n3 $n7 1.5Mb 10ms DropTail
$ns duplex-link $n4 $n5 1.5Mb 10ms DropTail
$ns duplex-link $n4 $n6 1.5Mb 10ms DropTail


set mproto DM
set mrthandle [$ns mrtproto $mproto {}]


set group1 [Node allocaddr]
set group2 [Node allocaddr]


set udp0 [new Agent/UDP]
$ns attach-agent $n0 $udp0
$udp0 set dst_addr_ $group1
$udp0 set dst_port_ 0
set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp0


set udp1 [new Agent/UDP]
$ns attach-agent $n1 $udp1
$udp1 set dst_addr_ $group2
$udp1 set dst_port_ 0
set cbr2 [new Application/Traffic/CBR]
$cbr2 attach-agent $udp1


set revr1 [new Agent/Null]
$ns attach-agent $n5 $revr1
$ns at 1.0 "$n5 join-group $revr1 $group1"

set revr2 [new Agent/Null]
$ns attach-agent $n6 $revr2
$ns at 1.5 "$n6 join-group $revr2 $group1"

set revr3 [new Agent/Null]
$ns attach-agent $n7 $revr3
$ns at 2.0 "$n7 join-group $revr3 $group1"


set revr4 [new Agent/Null]
$ns attach-agent $n5 $revr4
$ns at 2.5 "$n5 join-group $revr4 $group2"

set revr5 [new Agent/Null]
$ns attach-agent $n6 $revr5
$ns at 3.0 "$n6 join-group $revr5 $group2"

set revr6 [new Agent/Null]
$ns attach-agent $n7 $revr6
$ns at 3.5 "$n7 join-group $revr6 $group2"

$ns at 4.0 "$n5 leave-group $revr1 $group1"
$ns at 4.5 "$n6 leave-group $revr2 $group1"
$ns at 5.0 "$n7 leave-group $revr3 $group1"

$ns at 5.5 "$n5 leave-group $revr4 $group2"
$ns at 6.0 "$n6 leave-group $revr5 $group2"
$ns at 6.5 "$n7 leave-group $revr6 $group2"


$ns at 0.5 "$cbr1 start"
$ns at 9.5 "$cbr1 stop"

$ns at 0.5 "$cbr2 start"
$ns at 9.5 "$cbr2 stop"

$ns at 10.0 "finish"

proc finish {} {
global ns tf fd
$ns flush-trace
close $tf
close $fd
exec nam mcast.nam &
exit 0
}

$ns run
