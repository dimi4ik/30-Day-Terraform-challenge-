# Day 7: Understanding Terraform State.

## Participant Details
- **Name:** William Maina
- **Task Completed:** : Understanding Terraform State Part
- **Date and Time:** 2024-10-07 10:00pm EAT


### Configured remote state storage using S3 and locke the state using DynamoDB
#### main.tf
```hcl
provider "aws" {
  region = "us-east-1"
}
resource "aws_s3_bucket" "tfstate" {
  bucket = "terraformstate43210"

  lifecycle {
    prevent_destroy = true
  }
}
resource "aws_s3_bucket_versioning" "enable" {
  bucket = aws_s3_bucket.tfstate.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "server" {
  bucket = aws_s3_bucket.tfstate.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "AES256"
  }
}
}

resource "aws_s3_bucket_public_access_block" "block" {
  bucket = aws_s3_bucket.tfstate.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
resource "aws_dynamodb_table" "basic-dynamodb-table" {
  name           = "terraform_locks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
terraform {
  backend "s3" {
  bucket = "terraformstate43210"
  key = "globar/s3/terraform.tfstate"
  region = "us-east-1"

  dynamodb_table = "terraform_locks"
  encrypt = true
  }
}
```

