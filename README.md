

# Written By GeekTopia
#
# Copy RDS Automated snapshots to a new region upon creation
# --Free to use under all conditions
# --Script is provied as is. No Warranty, Express or Implied

import json
import boto3

destinationRegion = "us-east-1"

def lambda_handler(event, context):
    
    sourceRegion = event['region']
    
    rds = boto3.client('rds',region_name=destinationRegion)
  
    #Build the Snapshot ARN - Always use the ARN when copying snapshots across region. 
    sourceSnapshotARN = event['detail']['SourceArn']
    sourceSnapshotARN= sourceSnapshotARN.replace(":db:",":snapshot:")
    
    #build a new snapshot name
    sourceSnapshotIdentifer = event['detail']['SourceIdentifier']
    targetSnapshotIdentifer ="{0}-ManualCopy".format(sourceSnapshotIdentifer)
    targetSnapshotIdentifer = targetSnapshotIdentifer.replace(":","-")

    #Execute copy
    try:
        copy = rds.copy_db_snapshot(SourceDBSnapshotIdentifier=sourceSnapshotARN,TargetDBSnapshotIdentifier=targetSnapshotIdentifer,SourceRegion=sourceRegion)
        print("Started Copy of Snapshot {0} in {2} to {1} in {3} ".format(sourceSnapshotIdentifer,targetSnapshotIdentifer,sourceRegion,destinationRegion))
    
    except ClientError as ex:
        if ex.response['Error']['Code'] == 'DBSnapshotAlreadyExists':
            print("Snapshot  {0} already exist".format(targetSnapshotIdentifer))
        else:
            print("ERROR: {0}".format(ex.response['Error']['Code']))

    return {
        'statusCode': 200,
        'body': json.dumps('Opearation Complete')
    }   
                                            




{
    "source": [
        "aws.rds"
    ],
    "detail-type": ["RDS DB Snapshot Event"],
    "detail": {
       "Message":["Automated snapshot created"]
     }
}
