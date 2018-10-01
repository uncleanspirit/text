# text
{
	"Conditions": 
	{
		"EmptyKey": 
		{
			"Fn::Equals": 
			[
				{
					"Ref": "EC2KeyName"
				},

				""
			]
		},
		"EmptySUDORole":
		{
			"Fn::Equals": 
			[
				{
					"Ref": "AdGroupSudoAccess"
				},

				""
			]
		}
	},

	"Parameters":
	{
		"EFSTargetCname": 
		{
			"Type": "String"
		},

		"EFSTargetSG": 
		{
			"Type": "String"
		},

		"EC2KeyName": 
		{
			"Description": "This is the key pair that can be used to ssh onto the server",
			"Type": "String"
		},

		"AdGroupAccess": 
		{
			"Default": "",
			"Description": "This is a comma delimited list of the AD roles able to SSH to the ec2-instances",
			"Type": "String"
		},

		"AdGroupSudoAccess": 
		{
			"Default": "",
			"Description": "This is a comma delimited list of the AD roles able to sudo to the ec2-instances",
			"Type": "String"
		},

		"LogLevel": 
		{
			"Description": "This is the log level for the queing process",
			"Type": "String"
		},

		"VPCName": 
		{
			"Description": "The name of the VPC needed",
			"Type": "String"
		},

		"WebServerInstanceType": 
		{
			"Description": "This is the instance type of the web server",
			"Type": "String"
		},

		"InstanceProfileTemplateURL": 
		{
			"Description": "This is the update url of the instance profile cloud formation template",
			"Type": "String"
		},

		"CloudFormationBucketURL": 
		{
			"Description": "This is the URL to the cloudformation template",
			"Type": "String"
		},

		"ServerContentURL": 
		{
			"Description": "This is the URL to the server content to be deployed",
			"Type": "String"
		},

		"CertS3Bucket": 
		{
			"Description": "This is the S3 bucket for the cert files",
			"Type": "String"
		},
		"ApplicationPayloadBukcet":
		{
			"Description": "This is the S3 bucket for the application payload",
			"Type": "String"
		},
		"configBucket": 
		{
			"Description": "This is the S3 bucket for the config files",
			"Type": "String"
		},

		"S3fsBucket": 
		{
			"Description": "This is the S3 bucket that is leveraged by S3FS",
			"Type": "String"
		},

		"WebServerAsgDesiredCapacity": 
		{
			"Description": "This is the auto scaling group desired capacity for the web servers",
			"Type": "String"
		},

		"WebServerAsgMaxSize": 
		{
			"Description": "This is the auto scaling group max size for the web servers",
			"Type": "String"
		},

		"WebServerAsgMinSize": 
		{
			"Description": "This is the auto scaling group min size for the web servers",
			"Type": "String"
		},

		"ELBS3LoggingBucket": 
		{
			"Description": "This is the logging bucket that the elastic load balancer will use",
			"Type": "String"
		},

		"ApplicationName": 
		{
			"Description": "The name of our Application (can be some derivation of App ID)",
			"Type": "String"
		},

		"Version": 
		{
			"Description": "The version of the application",
			"Type": "String"
		},

		"Route53HostedZone": 
		{
			"Description": "This is the route53 hosted zone",
			"Type": "String"
		},

		"ClientAppPrefix": 
		{
			"Description": "The app prefix of the calling client app",
			"Type": "String"
		},

		"SysLevel": 
		{
			"AllowedValues": 
			[
				"eng",
				"test",
				"prd"
			],
			"Description": "This is the key pair that can be used to ssh onto the server (This should not have a value except in rare debugging occasions)",
			"Type": "String"
		},
		"AMIOSdistribution" : 
		{
			"Description" : "Name of the Operating system of the base AMI",
			"Type" : "String"
		},
		"AMIOSRelease" : 
		{
			"Description" : "Release of the Operating system for the Base AMI",
			"Type" : "String"
		},
		"AMIApplication" : 
		{
			"Description" : "Application Team creating the base AMI ",
			"Type" : "String"
		},
		"AMIRole" : 
		{
			"Description" : "Role of the Base AMI",
			"Type" : "String"
		},
		"RunSchedule" : 
		{
			"Description" : "Setup the run schedule for the EC2 instances",
			"Type" : "String"
		},
		"Owner" : 
		{
			"Description" : "Setup the Owner for the EC2 instances",
			"Type" : "String"
		},
		"SupportEmail" : 
		{
			"Description" : "Setup the run schedule for the EC2 instances",
			"Type" : "String"
		}
	},

	"Resources": 
	{
		"ODLaunchConfiguration": 
		{
		"Type": "AWS::AutoScaling::LaunchConfiguration",
			"DependsOn": 
			[
				"WebServerELB"
			],

			"Metadata": 
			{
				"Comment": "Launch Configuration for OpenDeploy receiver",
				"AWS::CloudFormation::Init": 
				{
					"configSets": 
					{
						"application_install": 
						[
							"security_updates"
						]
					},

					"security_updates": 
					{
						"commands": 
						{
							"ssh_ad_group": 
							{
								"command": "sed -i \"/-:ALL:ALL/i +:$adSshGroups:ALL\" /etc/security/access.conf",
								"env": 
								{
									"adSshGroups": 
									{
										"Ref": "AdGroupAccess"
									}
								}
							},

							"sudo": 
							{
								"Fn::If": 
								[
									"EmptySUDORole",
									{
										"Ref": "AWS::NoValue"
									},
									{
										"command": "echo $adSudoGroups | awk -F, '{ for (i=1;i<=NF; i++) print \"\\\"%\"$i\"\\\"  ALL=(ALL) PASSWD: ALL\" >> \"/etc/sudoers\" }'",
										"env": 
										{
											"adSudoGroups": 
											{
												"Ref": "AdGroupSudoAccess"
											}
										}
									}
								]
							}
						},

						"files": 
						{
							"/var/awslogs/etc/awslogs.conf": 
							{
								"content": 
								{
									"Fn::Join": 
									[
										"",
										[
											"[general]\n",
											"state_file = /var/awslogs/state/agent-state\n",
											"use_gzip_http_content_encoding = true\n",
											"\n",
											
											"[/opt/vgi/Interwoven/OpenDeployNG/log]\n",
											"file = /opt/vgi/Interwoven/OpenDeployNG/log/*\n",
											"log_group_name = OpenDeployLogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",
											
											"[/var/log/messages]\n",
											"file = /var/log/messages\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"\n",
											
											"[/var/log/daemonlog]\n",
											"file = /var/log/daemonlog\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",
											
											"[/var/log/kernlog]\n",
											"file = /var/log/kernlog\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",

											"[/var/log/authlog]\n",
											"file = /var/log/authlog\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",
											
											"[/var/log/cron]\n",
											"file = /var/log/cron\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",
											
											"[/var/log/syslog]\n",
											"file = /var/log/syslog\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",
											
											"[/var/log/sudolog]\n",
											"file = /var/log/sudolog\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %Y %H:%M:%S\n",
											"\n",
											
											"[/var/log/bootlog]\n",
											"file = /var/log/boot.log\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"\n",
											
											"[/var/log/cloud-init-log]\n",
											"file = /var/log/cloud-init.log\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %b %d %H:%M:%S\n",
											"\n",
											
											"[/var/log/cfn-init-cmd-log]\n",
											"file = /var/log/cfn-init-cmd.log\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %Y-%m-%d %H:%M:%S\n",
											"\n",
											
											"[/var/log/cfn-init-log]\n",
											"file = /var/log/cfn-init.log\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %Y-%m-%d %H:%M:%S\n",
											"\n",
																						
											"[/var/log/cloud-init-output-log]\n",
											"file = /var/log/cloud-init-output.log\n",
											"log_group_name = EC2Syslogs\n",
											"log_stream_name = {instance_id}\n",
											"datetime_format = %d %b %Y %H:%M:%S\n",
											"\n"

										]
									]
								}
							}
						}
					},

					"configure_application": 
					{
						"commands": 
						{
							"execute_odreceiver": 
							{
								"command": "su -c \"cd /cm/ansible; ansible-playbook /cm/ansible/main.yml --extra-vars \\\"$ansibleExtraVars\\\"\" -",
								"ignoreErrors": "false"
							}
						}
					}
				}
			},

			"Properties": 
			{
				"IamInstanceProfile": 
				{
					"Fn::Join": 
					[
						"",
						[
							"arn:aws:iam::",
							{
								"Ref": "AWS::AccountId"
							},

							":instance-profile/",
							{
								"Ref": "ODReceiverInstanceProfile"
							}
						]
					]
				},

				"ImageId": 
				{
					"Fn::GetAtt": 
					[
						"WebServerAMI",
						"image-id"
					]
				},

				"InstanceType": 
				{
					"Ref": "WebServerInstanceType"
				},

				"KeyName": 
				{
					"Fn::If": 
					[
						"EmptyKey",
						{
							"Ref": "AWS::NoValue"
						},

						{
							"Ref": "EC2KeyName"
						}
					]
				},

				"SecurityGroups": 
				[
					{
						"Ref": "WebServerSecurityGroup"
					},

					{
						"Fn::GetAtt": 
						[
							"LinuxBaseSg",
							"Id"
						]
					},

					{
						"Ref": "EFSTargetSG"
					}
				],

				"UserData": 
				{
					"Fn::Base64": 
					{
						"Fn::Join": 
						[
							"",
							[
								"#!/bin/bash\n",
								". /etc/export_proxy_vars\n",
								"export HTTP_PROXY=$PROXY_CONFIG\n",
								"export HTTPS_PROXY=$PROXY_CONFIG\n",
								"export NO_PROXY=\"$NOPROXY_CONFIG\"\n",
								"mkdir /var/log/ansible\n",
								"/usr/sbin/groupadd -g 9173 wcs \n",
								"/usr/sbin/adduser awcs001q --uid 9173 --gid 9173 \n",
								"mkdir -p /opt/vgi/Interwoven\n",
								"mkdir -p /data/vgi/opendeploy\n",
								"chown awcs001q.wcs /data/vgi/opendeploy\n",
								"yum install -y libstdc++.so.6 \n",
								"yum install -y libssl.so.6 \n",
								"growpart /dev/xvda 1\n",
								"export syslevel=",
								{
									"Ref": "SysLevel"
								},

								" CertS3Bucket=",
								{
									"Ref": "CertS3Bucket"
								},

								" region=",
								{
									"Ref": "AWS::Region"
								},

								" s3fsBucket=",
								{
									"Ref": "S3fsBucket"
								},

								"\n",
								"cfn-init -v ",
								"         --stack ",
								{
									"Ref": "AWS::StackName"
								},

								"         --resource ODLaunchConfiguration",
								"         --configsets application_install ",
								"         --region ",
								{
									"Ref": "AWS::Region"
								},

								"\n",
								"cfn-signal -e $? --stack ",
								{
									"Ref": "AWS::StackName"
								},

								"         --resource WebServerASG ",
								"         --region ",
								{
									"Ref": "AWS::Region"
								},

								"\n",
								"service awslogs restart\n",
								"echo ",
								{
									"Ref": "EFSTargetCname"
								},

								":/ /data/vgi/opendeploy efs tls,_netdev 0 0 >>/etc/fstab \n",
								"cd /opt/vgi/Interwoven\n",
								"sed -i \"s/stunnel_check_cert_hostname = true/stunnel_check_cert_hostname = false/\" /etc/amazon/efs/efs-utils.conf \n",
								"curl -O \"https://artifactory.opst.c1.vanguard.com/artifactory/vg-3rdparty-ics/opendeploy/iwovopendeployrcvr.linux.8.1.0.tar.gz\"\n",
								"mount /data/vgi/opendeploy\n",
								"mkdir /opt/vgi/installs\n",
								"cd  /opt/vgi/installs/\n",
								"/usr/bin/aws s3 sync ",
								{
									"Ref": "configBucket"
								},

								"/files/playbooks/files .\n",
                              	"/etc/init.d/update_hosts_file start \n",
                              	"/sbin/chkconfig update_hosts_file off \n",
                              	"unset DISPLAY \n",
                                "mkdir /tmp/odlogs \n",
                                "cp /opt/vgi/installs/clearLogs.pl /etc/cron.daily \n",
                                "/bin/chmod +x  /etc/cron.daily/clearLogs.pl \n",
								"mkdir /opt/vgi/installs/OD-expanded\n",
								"cd /opt/vgi/installs/OD-expanded\n",
								"/bin/tar xf /opt/vgi/Interwoven/iwovopendeployrcvr.linux.8.1.0.tar.gz \n",
								"/bin/chmod +x /opt/vgi/installs/updatehostsfile.sh\n",
								"/opt/vgi/installs/OD-expanded/startinstall_od < /opt/vgi/installs/od-receiver.txt\n",
								"/etc/init.d/update_hosts_file start \n",
								"/bin/bash /opt/vgi/installs/updatehostsfile.sh\n",
                                "cp /opt/vgi/installs/", 
                                {
                                    "Ref": "VPCName" 
                                },
                                "-OD.lic /opt/vgi/Interwoven/OpenDeployNG/etc/OD.lic \n",
                                "mkdir -p  /opt/vgi/Interwoven/OpenDeployNG/bin/demoCA/certs/ \n",
                                "mkdir -p  /opt/vgi/Interwoven/OpenDeployNG/cert/ \n",
                                "cp /opt/vgi/installs/odrcvr.xml /opt/vgi/Interwoven/OpenDeployNG/etc/odrcvr.xml \n",
                                "cp /opt/vgi/installs/cacert.pem /opt/vgi/Interwoven/OpenDeployNG/bin/demoCA/certs/cacert.pem \n",
                                "cp /opt/vgi/installs/odtgtcert.pem /opt/vgi/Interwoven/OpenDeployNG/cert/odtgtcert.pem \n",
                                "cp /opt/vgi/installs/odtgtkey.pem  /opt/vgi/Interwoven/OpenDeployNG/cert/odtgtkey.pem \n",
                                "cp /opt/vgi/installs/log4j.properties /opt/vgi/Interwoven/OpenDeployNG/etc/log4j.properties \n",
                                "/bin/bash /opt/vgi/installs/updateodrcvr.sh\n",
								"/etc/init.d/iwodserver stop\n",
								"chmod 777 /var/run/lock/subsys/ \n",
								"/opt/vgi/Interwoven/OpenDeployNG/bin/iwodnonroot awcs001q wcs  /opt/vgi/Interwoven/OpenDeployNG/ \n",
								"ln -s /etc/init.d/iwodserver /etc/init.d/iwodserver_boot \n",
								"cp /opt/vgi/installs/iwodserver_boot_wrap /etc/init.d/iwodserver_boot_wrap \n",
								"/bin/chmod +x /etc/init.d/iwodserver_boot_wrap\n",
								"rm /etc/rc5.d/S80iwodserver \n",
								"ln -s /etc/init.d/iwodserver_boot_wrap /etc/rc5.d/S80iwodserver \n",
								"/etc/init.d/iwodserver_boot_wrap start\n",
								"/usr/bin/updatedb\n"
							]
						]
					}
				}
			}

			
		},

		
		"WebServerAMI": 
		{
			"Properties": 
			{
				"name": "CCT-CentOS-7-*",
                "OS_Distribution": "CentOS",
                "OS_Release": "7",
                "Application": "UTS",
				"Region":
				{
					"Ref": "AWS::Region"
				},
				"Role": 
				{
					"Ref": "AMIRole"
				},
				"ServiceToken": 
				{
					"Fn::Join": 
					[
						"",
						[
							"arn:aws:lambda:",
							{
								"Ref": "AWS::Region"
							},

							":",
							{
								"Ref": "AWS::AccountId"
							},

							":function:AMILookupLambda"
						]
					]
				}
			},

			"Type": "Custom::AmiInfo"
		},
		"WebServerASG": 
		{
			"CreationPolicy": 
			{
				"ResourceSignal": 
				{
					"Count": 1,
					"Timeout": "PT15M"
				}
			},

			"DependsOn": 
			[
				"WebServerELB"
			],

			"Properties": 
			{
				"Cooldown": 1200,
				"DesiredCapacity": 
				{
					"Ref": "WebServerAsgDesiredCapacity"
				},

				"HealthCheckGracePeriod": 1000,
				"HealthCheckType": "ELB",
				"LaunchConfigurationName": 
				{
					"Ref": "ODLaunchConfiguration"
				},

				"LoadBalancerNames": 
				[
					{
						"Ref": "WebServerELB"
					}
				],

				"MaxSize": 
				{
					"Ref": "WebServerAsgMaxSize"
				},

				"MetricsCollection": 
				[
					{
						"Granularity": "1Minute"
					}
				],

				"MinSize": 
				{
					"Ref": "WebServerAsgMinSize"
				},

				"Tags": 
				[
					{
						"Key": "Name",
						"PropagateAtLaunch": true,
						"Value": 
						{
							"Fn::Join": 
							[
								"-",
								[
									{
										"Ref": "ApplicationName"
									},

									{
										"Ref": "Version"
									},

									"WebServer"
								]
							]
						}
					},

					{
						"Key": "NetSeg",
						"PropagateAtLaunch": true,
						"Value": "ServicesItCritical"
					},

					{
						"Key": "RunSchedule",
						"PropagateAtLaunch": true,
						"Value": 
						{
							"Ref" : "RunSchedule"
						}
					},
					{
						"Key": "Owner",
						"PropagateAtLaunch": true,
						"Value": 
						{
							"Ref" : "Owner"
						}
					},
					{
						"Key": "SupportEmail",
						"PropagateAtLaunch": true,
						"Value": 
						{
							"Ref" : "SupportEmail"
						}
					}
				],

				"VPCZoneIdentifier": 
				{
					"Fn::GetAtt": 
					[
						"SubnetId",
						"SubnetIds"
					]
				}
			},

			"Type": "AWS::AutoScaling::AutoScalingGroup"
		},

		"WebServerELB": 
		{
			"Type": "AWS::ElasticLoadBalancing::LoadBalancer",
			"Properties": 
			{
				"AccessLoggingPolicy": 
				{
					"EmitInterval": 5,
					"Enabled": true,
					"S3BucketName": 
					{
						"Ref": "ELBS3LoggingBucket"
					}
				},
				"HealthCheck":
				{
					"HealthyThreshold" : 10,
   					"Interval" : 120,
   					"Target" : "TCP:20014",
   					"Timeout" : 10,
   					"UnhealthyThreshold" : 2
				},
				"ConnectionDrainingPolicy": 
				{
					"Enabled": true,
					"Timeout": 1860
				},

				"ConnectionSettings": 
				{
					"IdleTimeout": 60
				},

				"CrossZone": "true",
				"Listeners": 
				[
					{
						"InstancePort": "20014",
						"InstanceProtocol": "TCP",
						"LoadBalancerPort": "20014",
						"Protocol": "TCP"
					}
				],

				"Scheme": "internal",
				"SecurityGroups": 
				[
					{
						"Ref": "WebServerELBSecurityGroup"
					}
				],

				"Subnets": 
				{
					"Fn::GetAtt": 
					[
						"SubnetId",
						"SubnetIds"
					]
				}
			}
		},

		"DnsHostedZoneId": 
		{
			"Properties": 
			{
				"ServiceToken": 
				{
					"Fn::Join": 
					[
						"",
						[
							"arn:aws:lambda:",
							{
								"Ref": "AWS::Region"
							},

							":",
							{
								"Ref": "AWS::AccountId"
							},

							":function:Route53ZoneLookup"
						]
					]
				},

				"DomainName": 
				{
					"Ref": "Route53HostedZone"
				},

				"PrivateZone": true
			},

			"Type": "Custom::Route53ZoneInfo"
		},

		"WebServerELBDNS": 
		{
			"Type": "AWS::Route53::RecordSetGroup",
			"DependsOn": 
			[
				"WebServerELB"
			],

			"Properties": 
			{
				"Comment": "This load balancer will receive a friendly DNS Name.",
				"HostedZoneId": 
				{
					"Fn::GetAtt": 
					[
						"DnsHostedZoneId",
						"ZoneId"
					]
				},

				"RecordSets": 
				[
					{
						"Type": "A",
						"AliasTarget": 
						{
							"DNSName": 
							{
								"Fn::GetAtt": 
								[
									"WebServerELB",
									"DNSName"
								]
							},

							"HostedZoneId": 
							{
								"Fn::GetAtt": 
								[
									"WebServerELB",
									"CanonicalHostedZoneNameID"
								]
							}
						},

						"Name": 
						{
							"Fn::Join": 
							[
								"",
								[
									{
										"Ref": "ClientAppPrefix"
									},

									"-",
									{
										"Ref": "SysLevel"
									},

									"-",
									{
										"Ref": "AWS::Region"
									},

									".",
									{
										"Ref": "Route53HostedZone"
									},

									"."
								]
							]
						}
					}
				]
			}
		},

		"SubnetId": 
		{
			"Type": "Custom::SubnetInfo",
			"Properties": 
			{
				"Await": "true",
				"ServiceToken": 
				{
					"Fn::Join": 
					[
						"",
						[
							"arn:aws:lambda:",
							{
								"Ref": "AWS::Region"
							},

							":",
							{
								"Ref": "AWS::AccountId"
							},

							":function:AwsSubnetLookup"
						]
					]
				},

				"SubnetNameBeginsWith": "Common"
			}
		},

		"WebServerELBSecurityGroup": 
		{
			"Properties": 
			{
				"GroupDescription": "WebServerELBSecurityGroup",
				"SecurityGroupIngress": 
				[
					{
						"CidrIp": "10.50.23.122/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.50.23.124/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.150.139.177/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.150.139.23/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.17.221.114/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.17.221.115/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.150.139.1/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.50.233.179/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					},
					{
						"CidrIp": "10.150.201.127/32",
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"ToPort": "20014"
					}
				],

				"VpcId": 
				{
					"Fn::GetAtt": 
					[
						"VpcId",
						"VpcId"
					]
				}
			},

			"Type": "AWS::EC2::SecurityGroup"
		},
		"ELBOutboundToODReceiver" : {
			"DependsOn": [
			"WebServerELBSecurityGroup",
			"WebServerSecurityGroup"
			],
			"Properties" : {
				"DestinationSecurityGroupId" : {
					"Ref" : "WebServerSecurityGroup"
				},
				"FromPort" : "20014",
				"GroupId": {
					"Fn::GetAtt": [
                        "WebServerELBSecurityGroup",
                        "GroupId"
                    ]
				
				},
				"IpProtocol" : "tcp",
				"ToPort": "20014"
			},
			"Type": "AWS::EC2::SecurityGroupEgress"
		},
		"WebServerSecurityGroup": 
		{
			"Properties": 
			{
				"GroupDescription": "WebServerSecurityGroup",
				"SecurityGroupIngress": 
				[
					{
						"FromPort": "20014",
						"IpProtocol": "tcp",
						"SourceSecurityGroupId": 
						{
							"Ref": "WebServerELBSecurityGroup"
						},

						"ToPort": "20014"
					}
				],

				"VpcId": 
				{
					"Fn::GetAtt": 
					[
						"VpcId",
						"VpcId"
					]
				}
			},

			"Type": "AWS::EC2::SecurityGroup"
		},

		"VpcId": 
		{
			"Properties": 
			{
				"ServiceToken": 
				{
					"Fn::Join": 
					[
						"",
						[
							"arn:aws:lambda:",
							{
								"Ref": "AWS::Region"
							},

							":",
							{
								"Ref": "AWS::AccountId"
							},

							":function:AwsVpcLookup"
						]
					]
				},

				"VpcName": 
				{
					"Ref": "VPCName"
				}
			},

			"Type": "Custom::VpcLookup"
		},

		"LinuxBaseSg": 
		{
			"Properties": 
			{
				"Await": "true",
				"ServiceToken": 
				{
					"Fn::Join": 
					[
						"",
						[
							"arn:aws:lambda:",
							{
								"Ref": "AWS::Region"
							},

							":",
							{
								"Ref": "AWS::AccountId"
							},

							":function:AwsSgLookup"
						]
					]
				},

				"sg": "LinuxBaseSg"
			},

			"Type": "Custom::SecurityGroupInfo"
		},

		"ODReceiverInstanceProfile": 
		{
			"Properties": 
			{
				"Roles": 
				[
					{
						"Ref": "ODReceiverApplicationRole"
					}
				]
			},

			"Type": "AWS::IAM::InstanceProfile"
		},

		"ODReceiverApplicationRole": 
		{
			"Properties": 
			{
				"AssumeRolePolicyDocument": 
				{
					"Statement": 
					[
						{
							"Action": 
							[
								"sts:AssumeRole"
							],

							"Effect": "Allow",
							"Principal": 
							{
								"Service": 
								[
									"ec2.amazonaws.com"
								]
							}
						}
					]
				},
				"Policies":
				[
					{
						"PolicyDocument":
						{
							"Statement": 
							[
		                		{
		                    		"Action": 
		                    		[
		                        		"s3:ListBucket"
		                    		],
		                    		"Effect": "Allow",
		                    		"Resource": "*"
		                		},
		                		{
		                			"Action": 
		                			[
		                        		"s3:GetObject"
		                    		],
		                    		"Effect": "Allow",
		                    		"Resource": 
		                    		[
		                        		{
		                        			"Fn::Join": 
		                        			[
		                        				"",
		                        				[
		                        					"arn:aws:s3:::",
		                        					{
		                        						"Ref": "ApplicationPayloadBukcet"	
		                        					},
		                        					"/*"
		                        				]
		                        			]
		                        		},
		                        		{
		                        			"Fn::Join": 
		                        			[
		                                		"",
		                                		[
		                                    		"arn:aws:s3:::",
		                                    		{
		                                        		"Ref": "CertS3Bucket"
		                                    		},
		                                    		"/*"
		                                		]
		                            		]
		                        			
		                        		}
		                        	]
		                        }
		                	],
		                	"Version": "2012-10-17"		
						},
						"PolicyName" : "S3BucketAccessInlinePolicy"
					}
				],
				"ManagedPolicyArns": 
				[
					{
						"Fn::Join": 
						[
							"",
							[
								"arn:aws:iam::",
								{
									"Ref": "AWS::AccountId"
								},

								":policy/CloudWatchLogsCustomerManaged"
							]
						]
					},
					{
						"Fn::Join": 
						[
							"",
							[
								"arn:aws:iam::",
								{
									"Ref": "AWS::AccountId"
								},

								":policy/KMSDecryptS3CustomerManaged"
							]
						]
					},
					{
						"Fn::Join": 
						[
							"",
							[
								"arn:aws:iam::",
								{
									"Ref": "AWS::AccountId"
								},

								":policy/DescribeMemberInstancesOfASGCustomerManaged"
							]
						]
					},
					{
						"Fn::Join":
						[
							"",
							[
								"arn:aws:iam::",
								{
									"Ref": "AWS::AccountId"
								},
								
								":policy/vgEc2BasePolicy"
							]
						]
					}
				]
			},
			"Type": "AWS::IAM::Role"
		}
	}
}
