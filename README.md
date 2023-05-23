# aws-backup-strategy
You can tag the different buckets, and create backup solution using those tags for resource selection

#Drop me know If you have any questions!

# s3bucket.tf

# Create an S3 bucket
resource "aws_s3_bucket" "example_bucket" {
  bucket = "example-bucket-tf-v1"
  tags = {
    "tier" = "bronze" #tag reference
  }  
}
resource "aws_s3_bucket_ownership_controls" "example" {
  bucket = aws_s3_bucket.example_bucket.id

  rule {
    object_ownership = "BucketOwnerPreferred"
    
  }
}

resource "aws_s3_bucket_versioning" "versioning_example" {
  bucket = aws_s3_bucket.example_bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}


# awsvault.tf

# Create an AWS Backup vault
# awsvault.tf

# Create an AWS Backup vault
resource "aws_backup_vault" "example_vault" {
  name        = "example-vault-tf"
}

data "aws_iam_policy_document" "example" {
  statement {
    effect = "Allow"

    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::24364625363765488878:user/jenny.system","arn:aws:iam::53635735735765656468:role/back-adm-role"]
    }

    actions = [
      "backup:DescribeBackupVault",
      "backup:DeleteBackupVault",
      "backup:PutBackupVaultAccessPolicy",
      "backup:DeleteBackupVaultAccessPolicy",
      "backup:GetBackupVaultAccessPolicy",
      "backup:StartBackupJob",
      "backup:GetBackupVaultNotifications",
      "backup:CopyIntoBackupVault",
      "backup:PutBackupVaultNotifications",
    ]

    resources = [aws_backup_vault.example_vault.arn]
  }
}

resource "aws_backup_vault_policy" "example" {
  backup_vault_name = aws_backup_vault.example_vault.name
  policy            = data.aws_iam_policy_document.example.json
}

resource "aws_backup_vault_lock_configuration" "test" {
  backup_vault_name   = aws_backup_vault.example_vault.name
  #changeable_for_days = 3
  max_retention_days  = 730
  min_retention_days  = 7
}



# awsbackup.tf

# Create an AWS Backup plan
resource "aws_backup_plan" "example_backup_plan" {
  name = "example-backup-plan-tf"

  rule {
    rule_name         = "example-backup-rule-daily"
    target_vault_name = aws_backup_vault.example_vault.name
    schedule          = "cron(0 5 * * ? *)"

    #lifecycle {
     # cold_storage_after = -1
   # }
   lifecycle {
      cold_storage_after = -1
      delete_after       = 14
    }

    copy_action {
      destination_vault_arn = aws_backup_vault.example_vault.arn
      lifecycle {
        cold_storage_after = -1
        delete_after       = 14
      }
      #copy_operation = "COPY"
    }

    
  }
  rule {
    rule_name         = "example-backup-rule-monthly"
    target_vault_name = aws_backup_vault.example_vault.name
    schedule          = "cron(0 5 1 * ? *)"

    
    lifecycle {
      cold_storage_after = 60
      delete_after                = 365
    }

    copy_action {
      destination_vault_arn = aws_backup_vault.example_vault.arn
      lifecycle {
        cold_storage_after = 60
        delete_after       = 365
      }
      
    }

    
  }

} 
resource "aws_backup_selection" "example_selection" {
    
      iam_role_arn = "arn:aws:iam::iam_role_arn = "arn:aws:iam::7566756568567787686:role/backup-admin-role"
name = "s3-root-bucket-resource-example"
plan_id = aws_backup_plan.example_backup_plan.id
#resources = [aws_s3_bucket.example_bucket.arn] no need to mention resource since we calling it from tags
selection_tag {

type = "STRINGEQUALS"
key = "backup_tier"
value = "bronze":role/backup-admin-role"
      name = "s3-root-bucket-resource-example"
      plan_id = aws_backup_plan.example_backup_plan.id
      
      selection_tag {
            type = "STRINGEQUALS"
            key = "backup_tier"
            value = "bronze"
    }
}



