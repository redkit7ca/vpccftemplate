{
    "AWSTemplateFormatVersion": "2010-09-09", //This line shows the AWS version
    "Description": "This template is for VPC",

    "Parameters": 
    { 
        // A specific KeyPair will be used as a paramater in this cloudformation.
        "KeyName": { 
            "Description": "This key is used for ssh into EC2 instance",
            "Type": "AWS::EC2::KeyPair::KeyName"
        },
        // A specific AMI (Amazon Machine Images) will be used as a paramater.
        "LatestAmiId": {
            "Type": "AWS::SSM:Parameter::Value<AWS::EC2::Image::Id>",
            "Default": "/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2"
        }
    },
    //This entire block declares the AWS resources which will be used.
    "Resources": 
    {
        //This block declares the "vpc1" settings.
        "vpc1": {
            "Type": "AWS::EC2::VPC", // This line shows the ASW resource type.
            "Properties": {
                "CidrBlock": "10.10.0.0/16",
                "EnableDnsHostnames" : true,
                "EnableDnsSupport" : true,
                "InstanceTenancy" : "default",
                "Tags": [ {"Key": "Name", "Value": "vpc1"} ]
            }
        },

        //This block declares the "vpc2" settings.
        "vpc2": {
            "Type": "AWS::EC2::VPC",
            "Properties": {
                "CidrBlock": "10.11.0.0/16",
                "EnableDnsHostnames" : true,
                "EnableDnsSupport" : true,
                "InstanceTenancy" : "default",
                "Tags": [ {"Key": "Name", "Value": "vpc2"} ]
            }
        },
        //This block declares "vpc1snA1" subnet settings.
        "vpc1snA1": {
            "Type": "AWS::EC2::Subnet",
            "Properties": {
                "VpcId": {"Ref": "vpc1"}, // With this line, "vpc1snA1" subnet assigns to vpc1.
                "Tags": [ {"Key": "Name", "Value": "vpc1_subnet_public"} ],
                "MapPublicIpOnLaunch" : true, //Whether to map the public IP on launch or not.
                
                //
                "AvailabilityZone" : {
                    //Returns a single object from a list of objects by index.
                    "Fn::Select": 
                    [
                        "0",//Returns us-east-1a as the first element of the array.
                        {
                            // Returns an array that lists Availability Zones for a specified region in alphabetical order.
                            "Fn::GetAZs": "" 
                        }
                    ]
                },
                "CidrBlock": "10.10.1.0/24"
            }
        },
        "vpc1snA2": {
            "Type": "AWS::EC2::Subnet",
            "Properties": {
                "VpcId": {"Ref": "vpc1"},
                "Tags": [ {"Key": "Name", "Value": "vpc1_subnet_private"} ],
                "MapPublicIpOnLaunch" : false,
                "AvailabilityZone" : {
                    "Fn::Select": [
                        "0",
                        {
                            "Fn::GetAZs": ""
                        }
                    ]
                },
                "CidrBlock": "10.10.2.0/24"
            }
        },
        
        //Internet Gateway (IGW) is being declared.
        "igwvpc1": 
        {
            "Type": "AWS::EC2::InternetGateway", // Resources type declared.
            "DependsOn": "vpc1", // This igw depends on vpc1 (must be created after vpc1)
            "Properties": {
                "Tags": [ { "Key": "Name", "Value": "vpc1igw"} ]
            }
        },
        
        "igwattachmentvpc1": {
            "Type": "AWS::EC2::VPCGatewayAttachment",
            "Properties": {
                "InternetGatewayId": {"Ref": "igwvpc1"},// igwvpc1 assingned to vpc1.
                "VpcId": {"Ref": "vpc1"}
            }
        }
    }
}
