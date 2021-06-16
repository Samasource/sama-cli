# Sama CLI

The Sama CLI helps you to create tasks in your Sama Project right from the terminal.

With the Sama CLI, you can:

- Create tasks in your Sama Project.
- Easily upload image, video, or 3d assets to Sama's S3 bucket.
- Generate a CVS or JSON of tasks to be created.
- Monitor task creation.

## Installation

Sama CLI is available for macOS, and Windows.

### macOS

Sama CLI is available on macOS via [Homebrew](https://brew.sh/):

```sh
brew tap Samasource/formulas
brew install sama
```
Note: You may need to preform `xcode-select --install` if you don't have xcode already installed.

### Windows

Sama CLI is available on Windows. Download the latest ".msi" file from the [release page](https://github.com/Samasource/sama-cli/releases/tag/v1.0.0) and open it to run the installer.

## Usage

Installing the CLI provides access to the `sama` command from the terminal.

Run the `sama` command to test if the CLI has been successfully installed:

```sh-session
$ sama
Sama CLI for interacting with the sama API

Usage:
  sama [flags]
  sama [command]

Available Commands:
  help        Help about any command
  init        initialises sama project, prompting the user for project settings.
  task        Root of task command tree

Flags:
  -h, --help      help for sama
  -v, --version   version for sama

Use "sama [command] --help" for more information about a command.
```

## Configuration

Before using the Sama CLI, you need to specify the Sama Project ID, API Key, and AWS credentials to Sama's S3 Bucket, all which are provided by your Sama project manager. The following instructions also assume that your Sama Project manager has already configured all the neccessary Sama Project inputs and outputs.

Create a new folder and set the current working directory to it. For example,
```sh
$ mkdir my_sama_project
$ cd my_sama_project
```

Run the `sama init` command:
```sh
$ sama init
? Project ID: YOUR_SAMA_PROJECT_ID
? API Key: YOUR_SAMA_API_KEY
? Sama AWS Access Key ID: YOUR_SAMA_AWS_ACCESS_KEY_ID
? Sama AWS Secret Access Key: YOUR_SAMA_AWS_SECRET_ACCESS_KEY
```
The above data will be saved into **.sama.yaml** in your current working directory.
Note: to use the AWS credentials stored in ~/.aws/credentials or %USERPROFILE%\\.aws\\credentials, leave “Sama AWS Access Key ID” and “Sama AWS Secret Access Key” blank.

You'll then be prompted to create the mapping json which contains your Sama Project inputs and maps it to values generated by the CLI. The CLI will _attempt to auto-generate_ the mapping json by looking at the required inputs set up in your Sama Project. See [Configuring the mapping json](#Configuring-the-mapping-json) below for more info.
```sh
? No mapping json found in project. Would you like to auto-generate it? Yes
{
  "name": "{{ .BaseName }}",
  "url": "{{ .URL }}",
  "client batch id": "{{ .ClientBatchID }}",
  "group id": "{{ .GroupID }}",
  "file size": "{{ .FileSizeBytes }}"
}
```
The above example has a Sama Project already set up with inputs "name"(string), "url"(URL), "client batch id"(string), "group id"(string), and "file size"(string). The mapping json will be saved into **.mapping.json.gotmpl** in your current working directory.

### Advanced configuration of the mapping json

The mapping json, **mapping.json.gotmpl**, contains your Sama Project inputs and values that will be sent during task creation. The values that you see in **{{ }}** are special variables used by the CLI and are replaced during task creation. The list of special variables are:
- `"{{ .BaseName }}"` - gets replaced by the asset's name, e.g. image001.png.
- `"{{ .URL }}"` - gets replaced by the asset's URL path in Sama's S3, e.g. https&#65279;://sama-client-assets.s3.amazonaws.com/123/assets_batch001/image001.png
- `"{{ .ClientBatchID }}"` - gets replaced by the string specified with the --client-batch-id flag
- `"{{ .GroupID }}"` - gets replaced by a unique groupID when using the --group-size flag
- `"{{ .FileSizeBytes }}"` - gets replaced by the asset's file size

When you call `sama init`, the CLI will do it's best to automatically use the special variables depending on your Sama Project's inputs.
- If you have an input called "name", "asset name", or "file name", the CLI will automatically insert "{{ .BaseName }}" as it's value.
- If you have an input type of "URL", the CLI will automatically insert "{{ .URL }}" as it's value.
- If you have an input type that is marked as "Client Batch ID", the CLI will automatically insert "{{ .ClientBatchID }}" as it's value.
- If you have an input type that is marked as "Task Group ID", the CLI will automatically insert "{{ .GroupID }}" as it's value.
- If you have an input called "size", "asset size", or "file size", the CLI will automatically insert "{{ .FileSizeBytes}}" as it's value.

If needed, modify **mapping.json.gotmpl** so that the proper data is sent during task creation. [Examples](#Examples) of the mapping json are given below.

## Commands

### `task create`
```
Name:
    task create

Description:
    Create a batch of tasks from files in a local directory or files in Sama's S3 Bucket.

Usage:
    sama task create <LocalPathFolder or S3UriFolder> [flags]

    LocalPathFolder: represents the path of a local folder.
    S3UriFolder: represents the location of a Sama S3 folder. This must be written in the form s3://sama-client-assets/myclientid/myfolder where myfolder can have additional subfolders
   
Flags:
      --client-batch-id string   client batch id value for mapping substitution
      --group-size int           will group tasks with this group size
  -h, --help                     help for create
      --in-batches               send each folder as its own batch
      --in-client-batches        send each folder as its own batch substituting the folder name as ClientBatchID
      --modified-after string    creates tasks from assets that were modified after given datetime string
      --modified-before string   creates tasks from assets that were modified before given datetime string
      --monitor                  if present will monitor the batch of tasks immediately after creation
      --output string            if present will output the task data to output file instead of sending task creation request
      --from-file                when true treats the path as local file path to a csv or json batch upload file
```

### `task monitor`

```sh
monitors the batch status in real time

Usage:
  sama task monitor <batch id> [flags]

Flags:
  -h, --help   help for monitor
```

## Examples

### Creating a batch of tasks given a local path
```
The mapping json can be automatically be generated during init. e.g.
    { 
        "name": "{{ .BaseName }}",
        "url": "{{.URL}}"
    }

The following 'task create' command creates a task for each video file. Assume the following asset folder exists locally:
    assets/
        assets_batch001/ 
            video001.mp4
            video002.mp4
            video003.mp4

$ sama task create assets/assets_batch001/
        
Output:
    A task will be created for video001.mp4:
        "name" : video001.mp4
        "url" : "https://sama-client-assets.s3.amazonaws.com/123/assets_001/video001.mp4"
    A task will be created for video002.mp4:
        "name" : video002.mp4
        "url" : "https://sama-client-assets.s3.amazonaws.com/123/assets_001/video002.mp4"
    A task will be created for video003.mp4:
        "name" : video003.mp4
        "url" : "https://sama-client-assets.s3.amazonaws.com/123/assets_001/video003.mp4"
```

### Creating a batch of tasks given a S3 path

```
The mapping json can be automatically be generated during init. e.g.
    { 
        "name": "{{ .BaseName }}",
        "url": "{{.URL}}"
    }

The following 'task create' command creates a task for each video file. Assume the following asset folder already exists in S3:
    s3://sama-client-assets/123/assets/assets_batch002/
        video001.mp4
        video002.mp4
        video003.mp4

$ sama task create s3://sama-client-assets/123/assets/assets_batch002/
        
Output:
    A task will be created for video001.mp4:
        "name" : video001.mp4
        "url" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch002/video001.mp4"
    A task will be created for video002.mp4:
        "name" : video002.mp4
        "url" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch002/video002.mp4"
    A task will be created for video003.mp4:
        "name" : video003.mp4
        "url" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch002/video003.mp4"
```


### Creating a batch of tasks that contains multiple URL inputs; for example, a 3D project with 2D helper videos

```
The mapping json will be automatically be generated during init as, for example:
    {
        "name": "{{ .BaseName }}", 
        "3d_url_input": "{{.URL}}",
        "3d_sensor_location" : "{{.URL}}",
        "2d_back_url_input": "{{.URL}}",
        "2d_front_url_input": "{{.URL}}"
    }

However, since there are multiple URL inputs, the folder or files containing the correct assets need to be specified. 
Folder example (don't forget to include the trailing "/" at the end of the folder names): 
    {
        "name": "{{ .BaseName }}", 
        "3d_url_input": "{{.URL}}lidar/",
        "3d_sensor_location" : "{{.URL}}sensor_location/",
        "2d_back_url_input": "{{.URL}}back_video/",
        "2d_front_url_input": "{{.URL}}front_video/"
    }
File example:
    {
        "name": "{{ .BaseName }}", 
        "3d_url_input": "{{.URL}}lidar.zip",
        "3d_sensor_location" : "{{.URL}}sensor_location.zip",
        "2d_back_url_input": "{{.URL}}back_video.mp4",
        "2d_front_url_input": "{{.URL}}front_video.mp4"
    }

The following 'task create' command creates a task for each folder which is a sequence. Assume the following asset folder exists locally (you would use the mapping json with the folder example above):
    assets/
        assets_batch001/
            sequence1/
                 lidar/    
                    001.pcd
                    002.pcd
                 back_video/
                    001.jpg
                    002.jpg
                 front_video/
                    001.jpg
                    002.jpg
                 sensor_location/
                    001.csv
                    002.csv
            sequence2/
                 lidar/    
                    001.pcd
                    002.pcd
                 back_video/
                    001.jpg
                    002.jpg
                 front_video/
                    001.jpg
                    002.jpg
                 sensor_location/
                    001.csv
                    002.csv

$ sama task create assets/assets_batch001

Output:
    A task will be created for sequence1/:
        "name" : sequence1
        "3d_url_input" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch001/sequence1/lidar/"
        "3d_sensor_location" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch001/sequence1/sensor_location/"
        "2d_back_url_input" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch001/sequence1/back_video/"
        "2d_front_url_input" : "https://sama-client-assets.s3.amazonaws.com/123/assets/assets_batch001/sequence1/front_video/"
   
    A task will be created for sequence2/:
        "name" : sequence2
        "3d_url_input" : "https://sama-client-assets.s3.amazonaws.com/123/assets_batch001/sequence2/lidar/"
        "3d_sensor_location" :"https://sama-client-assets.s3.amazonaws.com/123/assets_batch001/sequence2/sensor_location/"
        "2d_back_url_input" : "https://sama-client-assets.s3.amazonaws.com/123/assets_batch001/sequence2/back_video/"
        "2d_front_url_input" : "https://sama-client-assets.s3.amazonaws.com/123/assets_batch001/sequence2/front_video/"
```



### Monitoring the status of a batch immediately from task create command

```
Add the --monitor flag to 'task create':

$ sama task create assets_batch002 --monitor

```

### Monitoring the status of a batch

```
Assume that batch 11236 exists. Call 'task monitor':

$ sama task monitor 11236

Output:
 Batch 11236: 
  uploaded...

```

### Creating a batch of tasks and specifying a client batch id

```
The mapping json can be automatically be generated during init. e.g.
    { 
        "url": "{{.URL}}",
        "name": "{{ .BaseName }}",
        "client batch id": "{{ .ClientBatchID }}"
     }

The following 'task create' command creates task for each file. Assume the following assets exists locally:
    assets/
        assets_batch001/
            img001.png
            img002.png
            img003.png


$ sama task create assets/assets_batch001 --client-batch-id "my_batch_001"

Output:
    A batch will create individual tasks for img001.png, img002.png, img003.png and 'client batch id' is set to 'my_batch_001'.
```

### Creating multiple batch of tasks where each parent folder is considered a separate batch

```
The mapping file can be automatically be generated during init. e.g.
    { 
        "name": "{{ .BaseName }}",
        "url": "{{.URL}}"
    }

The following 'task create' command creates task for each file. Assume the following assets exists locally:
    assets/
        assets_batch001/
            img001.png
            img002.png
            img003.png
        assets_batch002/
            img004.png
            img005.png
            img006.png

$ sama task create assets --in-batches

Output:
    Batch 1 will create individual tasks for img001.png, img002.png, img003.png.
    Batch 2 will create individual tasks for img004.png, img005.png, img006.png.
```

### Creating multiple batch of tasks where each parent folder is considered a separate batch, and automatically assign the client batch id as the parent folder
```
The mapping file can be automatically be generated during init. e.g.
    { 
        "name": "{{ .BaseName }}",
        "url": "{{.URL}}",
        "client batch id": "{{ .ClientBatchID }}"
    }

The following 'task create' command creates task for each file. Assume the following assets exists locally:
    assets/
        assets_batch001/
            img001.png
            img002.png
            img003.png
        assets_batch002/
            img004.png
            img005.png
            img006.png

$ sama task create ./assets --in-client-batches

Output:
    A batch will create individual tasks for img001.png, img002.png, img003.png, and 'client batch id' is set to assets_batch001. The same batch will also create img004.png, img005.png, img006.png and 'client batch id' is set to assets_batch002.
```



### Create a batch of tasks and group them according to size

```
The mapping file can be automatically be generated during init. e.g.
    { 
        "example_input_url": "{{.URL}}",
        "name": "{{ .BaseName }}",
        "example_input_batch_id": "{{ .GroupID }}"
     }

The following 'task create' command creates task for each file. Assume the following assets exists locally:
    assets/
        assets_batch001/
            img001.png
            img002.png
            img003.png
            img004.png
            img005.png

$ sama task create assets/assets_batch001 --group-size 2

Output:
    Invididual tasks are created for img001.png, img002.png and are assigned to the same group id (auto assigned).
    Invididual tasks are created for img003.png, img004.png and are assigned to the same group id (auto assigned).
    Invididual tasks are created for img005.png and are assigned to the same group id (auto assigned).
```

### Creating an export of a batch of tasks to CSV or JSON file
```
$ sama task create ./assets_batch001 --output tasks.csv
$ sama task create ./assets_batch001 --output tasks.json
```

### Creating a batch of tasks from a CSV or JSON file
```
$ sama task create --from-file tasks.csv
$ sama task create --from-file tasks.json
```

### Creating a batch of tasks from assets that were modified within a date range
```
$ sama task create assets/assets_batch001 –modified-after 2021-06-10T00:00:00Z –modified-before 2021-06-24T00:00:00Z
```