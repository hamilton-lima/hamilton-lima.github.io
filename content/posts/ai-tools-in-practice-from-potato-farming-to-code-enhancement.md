---
title: 'AI Tools in Practice: From Potato Farming to Code Enhancement'
date: Thu, 07 Nov 2024 14:30:00 +0000
draft: false
tags: ['ai', 'coding', 'automation', 'learning', 'technology', 'future-of-work', 'potatoes']
---
![Maybe we should plant potatoes](/images/2024/ai-tools-in-practice-from-potato-farming-to-code-enhancement.png) 

In late 2022, when ChatGPT was launched, I remember joking with my colleagues that we should learn how to cultivate potatoes and beets because we'd soon be out of jobs. AI would do everything for us, right? Maybe that would be a future concern, something for generations down the road—it's hard to predict what's going to happen.

I know there's a lot of concern about implementing proper safeguards, preventing AI from taking control, and managing how it will change our jobs. There's also discussion about preparing people for new roles that will emerge with increased AI adoption. While these are important issues, they're not the problems I'm currently focused on solving. I'm observing these developments because I want to understand what people are thinking about it, but my current focus is different: How can I use AI to expand my technical capabilities?

Let me share some recent examples of exploring coding with AI support:

## Image Processing Investigation
I recently worked with some friends on detecting bubble quality in gold [froth flotation](https://en.wikipedia.org/wiki/Froth_flotation) process images. We needed to determine if the bubble formations were similar to so called good bubbles, to use this information to automate processes in the industrial pipeline. Though I hadn't done image processing in Python (which isn't my primary programming language) for a long time, I decided to give it a try using ChatGPT.

Through an iterative process of back-and-forth with the AI, copying and pasting code, I managed to build a working solution. What I realized was crucial: I knew what to ask. Even though Python wasn't my main language, I had enough knowledge to ask the right questions. I could guide the AI to help me separate concerns, build experiment reports, organize the code as a pipeline with single responsability concerns in mind and build artificial dataset to feed the Neural Network, you can see for yourself here: [machine-learning-sandbox](https://github.com/hamilton-lima/machine-learning-sandbox)

```python
def generate_rotated_images(input_image, input_folder, output_folder, num_versions=8, rotation_step=45):
    
    image = plt.imread(input_image)
    input_subfolder = os.path.relpath(os.path.dirname(input_image), input_folder)
    
    for version in range(num_versions):
        rotation_angle = version * rotation_step
        rotated_image = ndimage.rotate(image, rotation_angle)

        output_subfolder = os.path.join(output_folder, input_subfolder)
        os.makedirs(output_subfolder, exist_ok=True)
        
        output_filename = f"version_{version+1:02d}_{os.path.basename(input_image)}"
        output_path = os.path.join(output_subfolder, output_filename)
        
        # Normalize pixel values to the [0, 1] range
        rotated_image = (rotated_image - rotated_image.min()) / (rotated_image.max() - rotated_image.min()) * 255
        rotated_image = rotated_image.astype(np.uint8)
        
        plt.imsave(output_path, rotated_image)

        logging.info(f"Generated and saved: {output_path}")
```
Python image rotation code used to generate artificial trainning data by rotating multiple images and then generating multiple versions with different levels of brighteness.

### C# Technical Challenge
Last month, I faced a technical challenge for a job application that required C#. I hadn't touched C# in years, having spent over a decade primarily using Java, and the last 8 years using Typescript. While I could build the solution that was asked by referencing examples, and applying overal good practices in coding. sure some self promoting here... se my other article 😁[guiding-principles-for-cleaner-code](https://hamiltonlima.com/posts/guiding-principles-for-cleaner-code-yagni-kiss-solid-and-beyond/) I leveraged AI for creating unit tests. After implementing the solution, I took it a step further: I asked ChatGPT to analyze my code against a list of modern C# best practices. The AI provided valuable recommendations, some of which I implemented while others I chose to ignore based on my judgment.

The conclusion again: You have to know what to ask!

```C#
    private async Task<string> RunDockerCommand(string arguments)
    {
        var startInfo = new ProcessStartInfo
        {
            FileName = "docker",
            Arguments = $"run --rm {ImageName} {arguments}",
            RedirectStandardOutput = true,
            RedirectStandardError = true,
            UseShellExecute = false,
            CreateNoWindow = true
        };

        using var process = new Process { StartInfo = startInfo };
        process.Start();

        var output = await process.StandardOutput.ReadToEndAsync();
        var error = await process.StandardError.ReadToEndAsync();

        await process.WaitForExitAsync();

        if (process.ExitCode != 0)
        {
            throw new Exception($"Docker command failed with exit code {process.ExitCode}. Error: {error}");
        }

        // [HAL] This was really helpfull while building some of the tests 😊
        _output.WriteLine(output);
        return output.Trim();
    }
```
Fragment of the Integration test generated in C# to execute commands using a docker container. The original code recommended to build the image in the test process as well, but preffered to use an existing image to keep the focus on the problem at hand, the integration test itself.


### The Future of Programming
The challenge I see isn't about AI replacing programmers—it's about ensuring future programmers know what to ask. Perhaps the solution is that future developers will be able to accomplish much more, much faster, allowing them to be exposed to a broader range of problems. This could lead to exponential improvement in everyone's capabilities.
If I were teaching coding today, I'd likely present much more challenging problems—not because the fundamentals have changed, but because students would be EXPECTED to use AI as a tool. They could produce more substantial results in less time with less effort.

The key isn't just having AI tools—it's knowing how to use them effectively. Understanding the underlying concepts, knowing what questions to ask, and being able to evaluate the suggestions we receive will become increasingly crucial skills in our AI-augmented future.

PS. A good friend recommended we include Mint in the survival kit, it's good for digestion, so at the end it becomes: potatoes, beets and Mint, we may need it 😁🥔.
