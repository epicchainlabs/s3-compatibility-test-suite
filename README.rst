# S3 Compatibility Tests

This document provides a comprehensive guide to a series of unofficial Amazon AWS S3 compatibility tests. These tests are specifically designed to assist developers and engineers who are working on implementing software that exposes an S3-like API. By running these tests, you can ensure that your S3-compatible service meets the expected standards and functionalities. The tests leverage both the Boto2 and Boto3 libraries, making them versatile for different versions of your service.

The testing process is streamlined using the Tox tool, which facilitates the execution of test environments in an isolated manner. To begin, it’s essential to have the `tox` software installed on your system. If you are using a Debian-based system such as Ubuntu, you can install Tox easily with the following command:

```bash
sudo apt-get install tox
```

Before you run the tests, you need to create a configuration file that includes the details of your service’s endpoint and the credentials for two separate accounts. To help you get started, a sample configuration file named `s3tests.conf.SAMPLE` is provided within the repository. This file can be used as a template, particularly when running tests on a Ceph cluster initiated with vstart.

Once your configuration file is in place and customized according to your environment, you can initiate the testing process using the following command:

```bash
S3TEST_CONF=your.conf tox
```

You can further specify which directory of tests you wish to execute by using the following command:

```bash
S3TEST_CONF=your.conf tox -- s3tests_boto3/functional
```

If you prefer to run a specific test file, you can do so by executing:

```bash
S3TEST_CONF=your.conf tox s3tests_boto3/functional/test_s3.py
```

Moreover, if you want to run a particular test case within a file, you can narrow it down with:

```bash
S3TEST_CONF=your.conf tox s3tests_boto3/functional/test_s3.py::test_bucket_list_empty
```

In certain cases, tests are annotated with attributes that reflect their reliability or known issues, such as AWS not strictly enforcing its own specifications. You can filter these tests based on their attributes, allowing you to focus on tests that are relevant to your environment. For example, to exclude tests that are known to fail on AWS, you can run:

```bash
S3TEST_CONF=aws.conf tox -- -m 'not fails_on_aws'
```

The majority of these tests have versions written in both Boto2 and Boto3. Tests that utilize the Boto2 library are located in the `s3tests` directory, while those written for Boto3 can be found in the `s3test_boto3` directory. If you are only interested in running the Boto3 tests, you can do so with:

```bash
S3TEST_CONF=your.conf tox -- s3tests_boto3/functional
```

---

# STS Compatibility Tests

In addition to S3 tests, this document also covers compatibility tests for the Security Token Service (STS). These tests focus on three core APIs: AssumeRole, GetSessionToken, and AssumeRoleWithWebIdentity. The corresponding test cases are organized within the `s3tests_boto3/functional` directory.

Before running the STS tests, ensure that your vstart cluster is initiated with specific parameters to enable STS functionality. You can achieve this by including the following options when starting the cluster:

```bash
vstart.sh -o rgw_sts_key=abcdefghijklmnop -o rgw_s3_auth_use_sts=true
```

It is important to note that the `rgw_sts_key` parameter can be set to any value, as long as it is 128 bits in length. After your cluster is up and running, execute the following command to add the necessary capabilities to the test user:

```bash
radosgw-admin caps add --tenant=testx --uid="9876543210abcdef0123456789abcdef0123456789abcdef0123456789abcdef" --caps="roles=*"
```

To execute all the STS tests, which include tests for AssumeRole, GetSessionToken, and AssumeRoleWithWebIdentity, use the following command:

```bash
S3TEST_CONF=your.conf tox s3tests_boto3/functional/test_sts.py
```

If you wish to filter and run only specific tests, you can do so by applying filters based on the test attributes. For instance, to run only the AssumeRole and GetSessionToken tests, use the `test_of_sts` filter:

```bash
S3TEST_CONF=your.conf tox -- -m test_of_sts s3tests_boto3/functional/test_sts.py
```

For running the AssumeRoleWithWebIdentity tests, you will need to have Keycloak up and running in your environment. Additionally, make sure to include an "iam" section in your configuration file to support STS operations. You can refer to `s3tests.conf.SAMPLE` for guidance on how to structure your configuration file.

---

# IAM Policy Tests

This section focuses on testing IAM policies, particularly those that involve S3 actions. The tests cover a wide range of IAM-related scenarios, including the Put, Get, List, and Delete operations, as well as more complex policies with conflicting permissions.

The IAM policy tests are written using the Boto3 library and are located in the `s3test_boto3` directory. These tests rely on two user profiles, "iam" and "s3 alt," as defined in the `s3tests.conf.SAMPLE` file. If you are running your tests on a Ceph cluster started with vstart, these two user profiles will be automatically created with the same access keys, secret keys, and capabilities as specified in the configuration file.

The "iam" user profile is granted the capability `user-policy=*`, while the "s3 alt" user profile is created without such capabilities. These capabilities are configured during the vstart process if your Ceph cluster is started with vstart.

To run all the IAM policy tests, ensure that your configuration file includes sections for both "iam" and "s3 alt." You can then execute the tests with the following command:

```bash
S3TEST_CONF=your.conf tox s3tests_boto3/functional/test_iam.py
```

If you are interested in running a specific IAM policy test, you can specify it as follows:

```bash
S3TEST_CONF=your.conf tox s3tests_boto3/functional/test_iam.py::test_put_user_policy
```

As with the S3 and STS tests, some IAM policy tests have attributes that indicate known issues or specific behaviors. You can filter these tests using the `fails_on_rgw` attribute or other relevant tags:

```bash
S3TEST_CONF=your.conf tox -- s3tests_boto3/functional/test_iam.py -m 'not fails_on_rgw'
```

By following the instructions provided in this document, you can thoroughly test the compatibility and functionality of your S3-like API, ensuring that it meets the necessary standards and behaves as expected across different scenarios.

