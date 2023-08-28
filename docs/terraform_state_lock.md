# Terraform State Locking and How to Unlock State

## Problem Description

In a DevOps environment where Terraform is executed via GitHub Actions to manage infrastructure on AWS, there are instances where the state can become locked. This usually happens if the GitHub Action running the Terraform script is cancelled before Terraform can complete its operations. The state information, stored in AWS S3 and DynamoDB, becomes locked to prevent conflicts and inconsistencies, making it difficult to run subsequent Terraform commands.

## Why This Happens

Terraform uses state locking to protect the state from being modified by multiple users simultaneously. This feature is particularly useful when teams are working on the same infrastructure as it prevents any corruption or inconsistency in the state file.

In a setup where S3 is used for storing the Terraform state and DynamoDB for state locking, cancelling a GitHub Action in the middle of its execution interrupts the Terraform process, but it might not release the lock on the state file. This makes the state inaccessible for future runs or any other operations, requiring manual intervention to unlock it.

## Solution 1: Unlock Using Terraform CLI

You can use the `terraform force-unlock` command to manually unlock the state file.

### Steps

1. **Identify Lock ID**: First, identify the lock ID that is locking the state file. This is usually printed in the error message when you attempt a new Terraform operation that requires the state file.

2. **Run Force Unlock Command**: Execute the following command to unlock the state:

    ```bash
    terraform force-unlock [LOCK_ID] [path-to-terraform-config]
    ```

    - `[LOCK_ID]`: Replace with the actual Lock ID.
    - `[path-to-terraform-config]`: Replace with the path where your `terraform.tfstate` file resides.

3. **Verify**: After running the command, you should see a message indicating that the lock has been removed. You can now run Terraform operations as usual.

### Important

The `terraform force-unlock` command is a dangerous operation and should only be used when you are certain that no other ongoing Terraform operations are being run on the state file.

## Solution 2: Unlock Via DynamoDB

You can also manually remove the lock from the DynamoDB table that is being used for state locking.

### Steps

1. **Go to AWS Console**: Log into the AWS Management Console and navigate to DynamoDB.

2. **Locate the Table**: Locate the DynamoDB table used for Terraform state locking. This is usually configured in your Terraform `backend` configuration.

3. **Find Lock ID**: Search for the lock ID item in the DynamoDB table. It's typically stored as an item with the attribute `LockID`.

4. **Delete Item**: Select the item and delete it.

5. **Verify**: Make sure that the item is successfully deleted from the DynamoDB table. Your state should now be unlocked.

### Important

Directly manipulating the DynamoDB table to remove a lock is a risky operation and should only be done when you are absolutely sure that no other operations are being performed on the state file.

---

By following either of the two methods, you should be able to successfully unlock a Terraform state file that has been locked due to a canceled GitHub Action. Always exercise caution when performing these operations to avoid any corruption or loss of data.
