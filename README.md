# CloudFormation
Scripting Projects for AWS


AWSTemplateFormatversion: '2010-09-09'
Description: Julian-AWSWorkspaces-deployment-template

Parameters:

  Bundle:
    Type: String
    Description: Select the bundle that should be deployed
  Directory:
    Type: String
    Description: Enter the Directory ID for the WorkSpace (Directory Services or AD Connector)
  User:
    Type: String
    Description: Enter the username who will use the WorkSpace
    AllowedPattern: ^[a-z0-9]{1,20}$
    ConstraintDescription: user must be entered using lower case letters and numbers and a maximum of 20 characters
  IsRootVolumeEncrypted:
    Type: String
    Description: Inform if Root Volume will be encrypted
    AllowedValues:
      - true
      - false
    Default: "true"  
  IsUserVolumeEncrypted:
    Type: String
    Description: Inform if User Volume will be encrypted
    AllowedValues:
      - true
      - false
    Default: "true"       
  EncryptionKey:
    Type: String
    Description: Select the KMS encryption key to encrypt WorkSpace volumes
  RunningModeType:
    Type: String
    Description: Enter Running Mode
    AllowedValues:
      - ALWAYS_ON
      - AUTO_STOP  
    Default: "AUTO_STOP"

Conditions:
  HasEncryption: !Or [!Equals [!Ref "IsRootVolumeEncrypted", true], !Equals [!Ref "IsUserVolumeEncrypted", true]]
  NoEncryption: !And [!Equals [!Ref "IsRootVolumeEncrypted", false], !Equals [!Ref "IsUserVolumeEncrypted", false]]

Resources:

  WorkSpaceWithEncryption:
    Type: AWS::WorkSpaces::Workspace
    Condition: HasEncryption
    Properties:
      BundleId: !Ref Bundle
      DirectoryId: !Ref Directory
      UserName: !Ref User
      RootVolumeEncryptionEnabled: !Ref IsRootVolumeEncrypted
      UserVolumeEncryptionEnabled: !Ref IsUserVolumeEncrypted
      VolumeEncryptionKey: !If [HasEncryption, !Ref EncryptionKey, ""]
      WorkspaceProperties:
        RunningMode: !Ref RunningModeType

  WorkSpaceWithoutEncryption:
    Type: AWS::WorkSpaces::Workspace
    Condition: NoEncryption    
    Properties:
      BundleId: !Ref Bundle
      DirectoryId: !Ref Directory
      UserName: !Ref User
      RootVolumeEncryptionEnabled: !Ref IsRootVolumeEncrypted
      UserVolumeEncryptionEnabled: !Ref IsUserVolumeEncrypted
      WorkspaceProperties:
        RunningMode: !Ref RunningModeType

Outputs:
  WorkSpaceId:
    Description: ID of the WorkSpaces
    Value: !If [HasEncryption, !Ref WorkSpaceWithEncryption, !Ref WorkSpaceWithoutEncryption]  
