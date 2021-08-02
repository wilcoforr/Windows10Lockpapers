# Windows 10 Lockpapers

Probably being a bit of a bad boy about these various "lockpapers" copyrights :)

![Test Image](e0ce.jpg)


```csharp
namespace Windows10LockpaperDownloader
{
    using System;
    using System.Drawing;
    using System.IO;
    using System.Linq;

    static class Program
    {
        //the magic string that Windows 10 stores images at
        //To test this out, copy a "large" file above like 500 KB then add .jpg at the end of it and you will see an image.
        //this string path is like this:
        //  "C:\Users\MYCOMPUTERNAME\AppData\\Local\Packages\Microsoft.Windows.ContentDeliveryManager_cw5n1h2txyewy\LocalState\Assets\"
        static readonly string windows10LockWallaperPath = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData)
                    .Replace("Roaming", "")
                    + @"\Local\Packages\Microsoft.Windows.ContentDeliveryManager_cw5n1h2txyewy\LocalState\Assets\";

        static readonly string myPicturesPath = Environment.GetFolderPath(Environment.SpecialFolder.MyPictures);
        static readonly string picturesWindows10WallpapersPath = myPicturesPath + @"\Windows10LockWallpapers\";
        static readonly string landscapePath = picturesWindows10WallpapersPath + @"Landscape\";
        static readonly string portraitPath = picturesWindows10WallpapersPath + @"Portrait\";

        static void Main(string[] args)
        {
            try
            {
                CreateDirectoriesIfTheyDontExist();

                //get all files larger than 50 KB because these are likely not images, or just really small/poor quality
                var files = Directory.GetFiles(windows10LockWallaperPath)
                    .Select(f => new FileInfo(f))
                    .Where(f => f.Length > 50 * 1024);


                //it seems like even though the names are some sort of hex/base64 garbage, the names, or at least the first four letters, are the
                //same for duplicate images. thus:
                //  create a list of all file names, substring by 4
                //  for the image to be copied from the appdata: if that filename exists in pictures folder, dont copy

                var existingLandscapeFileNames = Directory.GetFiles(landscapePath)
                                                            .Select(f => new FileInfo(f).Name.Substring(0, 4));

                var existingPortraitFileNames = Directory.GetFiles(portraitPath)
                                                            .Select(f => new FileInfo(f).Name.Substring(0, 4));

                foreach (var file in files)
                {
                    string newImageFileName = file.Name.Substring(0, 4);

                    if (existingLandscapeFileNames.Contains(newImageFileName) || existingPortraitFileNames.Contains(newImageFileName))
                    {
                        Console.WriteLine("Already exists! Name: " + newImageFileName + ".jpg");
                    }
                    else
                    {
                        //place the image in either the landscape or portrait folder depending on its dimensions
                        //if Width greater than Height - landscape image
                        using (var img = new Bitmap(file.FullName))
                        {
                            if (img.Width > img.Height)
                            {
                                Console.WriteLine("Created new landscape image.");
                                File.Copy(file.FullName, landscapePath + newImageFileName + ".jpg");
                            }
                            else
                            {
                                Console.WriteLine("Created new portrait image.");
                                File.Copy(file.FullName, portraitPath + newImageFileName + ".jpg");
                            }
                        }
                    }
                }

                Console.WriteLine("Done creating images.");
                Console.WriteLine("Now opening pictures path: " + picturesWindows10WallpapersPath + " . . .");
                System.Diagnostics.Process.Start(picturesWindows10WallpapersPath);
            }
            catch (Exception ex)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine(ex);
                Console.ForegroundColor = ConsoleColor.White;
            }
            finally
            {
                Console.WriteLine(new string('-', 40));
                Console.WriteLine("Press enter to exit.");
                Console.ReadLine();
            }
        }

        static void CreateDirectoriesIfTheyDontExist()
        {
            if (!Directory.Exists(picturesWindows10WallpapersPath))
            {
                Directory.CreateDirectory(picturesWindows10WallpapersPath);
            }

            if (!Directory.Exists(landscapePath))
            {
                Directory.CreateDirectory(landscapePath);
            }

            if (!Directory.Exists(portraitPath))
            {
                Directory.CreateDirectory(portraitPath);
            }
        }

    }
}


```
