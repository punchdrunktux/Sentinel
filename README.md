# Sentinel


#Parse Sysmon in Sentinel
// Azure Sentinel Sysmon Basic Parse via KQL
Event
| where Source == "Microsoft-Windows-Sysmon"
| extend RenderedDescription = tostring(split(RenderedDescription, ":")[0])
| extend EventData = parse_xml(EventData).DataItem.EventData.Data
| mv-expand bagexpansion=array EventData
| evaluate bag_unpack(EventData)
| extend Key=tostring(['@Name']), Value=['#text']
| evaluate pivot(Key, any(Value), TimeGenerated, Source, EventLog, Computer, EventLevel, EventLevelName, EventID, UserName, RenderedDescription, MG, ManagementGroupName, Type, _ResourceId)
| extend RuleName = column_ifexists("RuleName", ""), TechniqueId = column_ifexists("TechniqueId", ""),  TechniqueName = column_ifexists("TechniqueName", "")
| parse RuleName with * 'technique_id=' TechniqueId ',' * 'technique_name=' TechniqueName
//| project TimeGenerated, Source, EventLog, Computer, EventLevel, EventLevelName, EventID, UserName, RenderedDescription, MG, ManagementGroupName, Type, _ResourceId, Archived, CallTrace, CommandLine, Company, CreationUtcTime, CurrentDirectory, Description, DestinationHostname, DestinationIp, DestinationIsIpv6, DestinationPort, DestinationPortName, Details, Device, EventType, FileVersion, GrantedAccess, Hashes, Image, ImageLoaded, Initiated, IntegrityLevel, IsExecutable, LogonGuid, LogonId, NewThreadId, OriginalFileName, ParentCommandLine, ParentImage, ParentProcessGuid, ParentProcessId, PipeName, PreviousCreationUtcTime, ProcessGuid, ProcessId, Product, Protocol, QueryName, QueryResults, QueryStatus,   RuleName, Signature, SignatureStatus, Signed, SourceHostname, SourceImage, SourceIp, SourceIsIpv6, SourcePort, SourcePortName, SourceProcessGuid, SourceProcessGUID, SourceProcessId,  SourceThreadId, StartAddress, StartFunction, StartModule, TargetFilename, TargetImage, TargetObject, TargetProcessGuid, TargetProcessGUID, TargetProcessId, TerminalSessionId, User,  UtcTime, TechniqueId, TechniqueName
| order by TimeGenerated desc 

Ref:

https://raw.githubusercontent.com/eshlomo1/Azure-Sentinel-4-SecOps/master/Sysmon/ParseSysmon.txt
