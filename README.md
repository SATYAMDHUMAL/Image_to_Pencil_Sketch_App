# Image_to_Pencil_Sketch_App
import java.awt.Color;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;

public class PencilSketch {
    public static void main(String[] args) {
        try {
            // Read the image
            BufferedImage image = ImageIO.read(new File("input_image.jpg"));

            // Convert image to pencil sketch
            BufferedImage pencilSketch = convertToPencilSketch(image);

            // Save the sketch to a file
            File output = new File("pencil_sketch_output.jpg");
            ImageIO.write(pencilSketch, "jpg", output);

            System.out.println("Pencil sketch created successfully!");
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    // Method to convert image to pencil sketch
    public static BufferedImage convertToPencilSketch(BufferedImage image) {
        int width = image.getWidth();
        int height = image.getHeight();

        BufferedImage grayscale = new BufferedImage(width, height, BufferedImage.TYPE_BYTE_GRAY);
        BufferedImage inverted = new BufferedImage(width, height, BufferedImage.TYPE_BYTE_GRAY);
        BufferedImage pencilSketch = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        // Convert image to grayscale
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                Color color = new Color(image.getRGB(x, y));
                int grayscaleValue = (int) (0.299 * color.getRed() + 0.587 * color.getGreen() + 0.114 * color.getBlue());
                grayscale.setRGB(x, y, new Color(grayscaleValue, grayscaleValue, grayscaleValue).getRGB());
            }
        }

        // Invert the grayscale image
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                int grayscaleValue = new Color(grayscale.getRGB(x, y)).getRed();
                int invertedValue = 255 - grayscaleValue;
                inverted.setRGB(x, y, new Color(invertedValue, invertedValue, invertedValue).getRGB());
            }
        }

        // Apply edge detection to the inverted image
        int[][] edgeMatrix = {
                {-1, -1, -1},
                {-1, 8, -1},
                {-1, -1, -1}
        };

        int edgeOffset = edgeMatrix.length / 2;

        for (int y = edgeOffset; y < height - edgeOffset; y++) {
            for (int x = edgeOffset; x < width - edgeOffset; x++) {
                int sum = 0;

                for (int i = 0; i < edgeMatrix.length; i++) {
                    for (int j = 0; j < edgeMatrix[i].length; j++) {
                        int pixelValue = new Color(inverted.getRGB(x + i - edgeOffset, y + j - edgeOffset)).getRed();
                        sum += edgeMatrix[i][j] * pixelValue;
                    }
                }

                int newValue = Math.min(Math.max(sum, 0), 255);
                pencilSketch.setRGB(x, y, new Color(newValue, newValue, newValue).getRGB());
            }
        }

        return pencilSketch;
    }
}
