# Using Application Icons from AWS

Previously the icons used over the COLID and Data Marketplace Websites have been stored as '.svg' images.
The current process has been automated where the .svg icons can be uploaded to an Amazon Web Services S3 bucket as per developers choice and it will be automatically reflected on the websites. 

## Amazon Simple Storage Service (Amazon S3)

Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. Amazon S3 is designed for 99.999999999% (11 9's) of durability, and stores data for millions of applications for companies all around the world.
The purpose of automating and fetching the application icons/images from AWS has multiple benefits.
* Benefits
  * Simple Data Transfer
  * Easy to Manage
  * Availability
  * Security
  * Availability  

## Strategy

The strategy for creating this solution has been achieved in the following steps.
1. Selecting and Saving the desired icons to the GitLab project repository.
2. Icons are uploaded to the S3 bucket via GitLab CI.
3. Create reusable npm package *@colid/icons* to fetch and display the icon from S3 Bucket.

## Details

*  Saving Icons to the GitLab repository
     - Project **COLID/client-apps/icon-upload** has been created to automatically upload the icons to AWS
     - The icon name should be saved by the user as the *encoded identifier of the resource type*. This naming pattern is an encoded string of the URI for frontend purposes and has to be renamed before uploading. 
     - A possible encoder can be found [by clicking here](https://meyerweb.com/eric/tools/dencoder/)
     - Example: Currently used on Resource Type Icons
        * Original filename: `Dataset.svg`
        * After adding URI: `http://pid.bayer.com/kos/19014/Dataset.svg`
        * After encoding: `http%3A%2F%2Fpid.bayer.com%2Fkos%2F19014%2FDataset.svg`  
     - Icons must be saved in the *icons/* folder of the project.

*  Uploading icons to the S3 bucket.
     - Once the icon is uploaded to the project repository, the Gitlab CI uses the *tgip/terraform-eks-tools:k8s1.13.7-helm2.11.0-tf0.12.6* docker image, which has all the tools installed to upload the icons to S3.
     - Icons (in svg format) are usually in sync with the external AWS S3 bucket and only used by the master branch. Other branches are ignored because icons are globally available. In case of adding, editing or removal of icons, all operations will be also processed on AWS S3 in the bucket `s3://dataservices-icons/`.
     - The CI Pipeline takes care of switching the AWS roles and authentication via valid Access and Secret Keys and Security Tokens thus maintaining the security and restricting any third-parties to update/view client-specific icons/images. 

*  npm package *@colid/icons*
    - A custom reusable npm package is built which fetches the icons from the S3 bucket.
    - The package uses custom *ColidIconsService* service and is currently being used to display resource type icons on following components
        * deletion-request component
        * resource-display component
        * resource-hierarchy component
        * sidebar-content component
