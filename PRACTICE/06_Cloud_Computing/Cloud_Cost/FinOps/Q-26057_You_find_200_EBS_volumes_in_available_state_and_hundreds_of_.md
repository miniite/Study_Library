---
type: "practice"
difficulty: "Intermediate"
---

# You find 200 EBS volumes in 'available' state and hundreds of old snapshots. Write the approach — how you'd identify the truly-orphaned ones safely and automate the cleanup without deleting something someone needs.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retention Costs and Compliance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to identify and clean up orphaned EBS volumes and snapshots in an AWS environment while ensuring that no necessary resources are accidentally deleted. The key is to automate the process safely by identifying idle or abandoned resources.

#### Triaging the Question

The interviewer is probing your understanding of cloud cost governance (FinOps) principles, your ability to manage AWS services effectively, and your expertise in automating resource management tasks. We need to make sure that we can reliably identify orphaned resources and clean them up without causing disruptions.

#### Assumptions
and Approach

#### Assumptions

The EBS volumes and snapshots are part of a production environment.
There might be legitimate reasons for keeping these resources, such as backups or ongoing development activities.
We need to avoid deleting any resources that are still in use.

Steps to Identify Orphaned Resources:

Step 1: Gather Information
List all EBS volumes and snapshots: Use AWS CLI commands or the AWS Management Console to list all available EBS volumes and snapshots.
BASH
# Example using AWS CLI
aws ec2 describe-volumes --filters Name=state,Values=available --query 'Volumes[*]' --output json > ebs_volumes.json

aws ec2 describe-snapshots --filters Name-status,Values=completed --query 'Snapshots[*]' --output json > snapshots.json
Step 2: Cross-Reference Usage Information
Check for attachments: Determine which EBS volumes are currently attached to EC2 instances.
BASH
aws ec2 describe-instances --query "Reservations[].Instances[?rootDeviceType=='ebs'].RootDeviceName" --output json > attached_volumes.json
Identify recent usage of snapshots: Check for any recent use or dependencies.
Step 3: Determine Idle Resources
Analyze the age and usage patterns:
Snapshots older than a certain threshold (e.g., 90 days) might be candidates.
BASH
import json

with open('snapshots.json') as f:
    snapshots = json.load(f)
orphan_snapshots = [s for s in snapshots if 'Tags' not in s or not any(tag['Value'] == 'Production' for tag in s.get('Tags', [])) and (datetime.now() - datetime.fromisoformat(s['StartTime']).date()).days > 90]

print(orphan_snapshots)
Identify volumes that are not attached to running instances:
BASH
import json

with open('ebs_volumes.json') as f, open('attached_volumes.json') as g:
    ebs_volumes = json.load(f)
    attached_volumes = {v['Attachments'][0]['InstanceId'] for v in json.load(g)}

orphan_volumes = [v for v in ebs_volumes if 'Attachments' not in v or len(v.get('Attachments', [])) == 0 and v['VolumeId'] not in attached_volumes]
Step 4: Prioritize Cleanup
Prioritize cleanup based on last usage date:
BASH
# Sort snapshots by creation time (oldest first)
orphan_snapshots.sort(key=lambda s: datetime.fromisoformat(s['StartTime']))

# Sort volumes by last attachment or creation time (oldest first)
orphan_volumes.sort(key=lambda v: datetime.fromisoformat(v.get('Attachments', [{}])[0].get('AttachmentTime', v['CreateTime'])))
Automating the Cleanup

To automate this process, we can write a script that performs the following steps:

Identify Orphans: Run the above scripts to gather and filter out idle resources.
Dry Run: Print out which resources would be deleted before actually deleting them.
BASH
# Example dry run script
echo "Dry-run: The following snapshots and volumes will be deleted:"
for s in "${orphan_snapshots[@]}"; do echo "Snapshot: ${s['SnapshotId']}, Created at: ${s['StartTime']}"; done

for v in "${orphan_volumes[@]}"; do echo "Volume: ${v['VolumeId']}, State: ${v['State']}, Last Attached Time: ${v['Attachments'][0]['AttachmentTime']}"; done
Confirm Deletion: If the dry run is confirmed, execute the deletion commands.
BASH
for s in "${orphan_snapshots[@]}"; do echo "Deleting Snapshot: ${s['SnapshotId']}..."; aws ec2 delete-snapshot --snapshot-id ${s['SnapshotId']}; done

for v in "${orphan_volumes[@]}"; do echo "Detaching and Deleting Volume: ${v['VolumeId']}..."; aws ec2 detach-volume --volume-id ${v['VolumeId']} --instance-id $(echo $v['Attachments'][0]['InstanceId']); sleep 15; aws ec2 delete-volume --volume-id ${v['VolumeId']}; done

#### Closing Summary
and Common Mistakes

By following this structured approach, we can systematically identify and clean up idle EBS volumes and snapshots. The key is to balance between cost reduction and maintaining operational integrity.

#### Common Mistakes

Failing to cross-reference usage: Not checking if a snapshot or volume is attached or has recent activity.
Overly aggressive deletion: Removing resources that are still in use, leading to service disruptions.
Lack of automation: Manually identifying and deleting resources increases the risk of human error.
Final Check

This approach ensures that we can safely clean up idle EBS volumes and snapshots without causing operational issues. The scripts provided are designed to be robust and flexible enough to handle various scenarios.

