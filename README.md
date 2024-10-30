# Extennsions for the ENGR 1181 Software Design Project

## Saving and loading a game state across runs

```MATLAB
% save variables
save(filename.mat, variables)

% load variables
load filename.mat
```

## Integrating text neatly

In general, text can be added with the [text command](https://www.mathworks.com/help/matlab/ref/text.html), but that displays with
a very different style and sizing than the board. Additionally, each time you use it you have to test where it ends up.

A cleaner way of doing this is with the [ascii.png](./ascii.png) spritesheet. It includes sprites for many common characters 
at the indices that correspond to ASCII. This means that text can be directly inserted into your board without having to worry 
about converting the sprite values back and forth between the characters they represent. You can include your own sprites in any 
spaces where you don't intend to use those characters.

Try this example script once you have ascii.png downloaded:

```MATLAB
symbols = simpleGameEngine("ascii.png",16,16,2,[150,0,0]);

screen = ['                       ';
          ' This sheet is aligned ';
          ' with ascii for a few  ';
          ' hundred characters,   ';
          ' so text can easily    ';
          ' just be printed       ';
          ' without having to     ';
          ' deal with using       ';
          ' constant values for   ';
          ' each character.       ';
          '                       '];

drawScene(symbols, screen);
```

The output should look like this:
![ASCII example output](./ascii-demo-output.png)

You may run into some issues with character arrays not automatically turning into integers.

To fix this, edit these lines of the SimpleGameEngine (lines 123-126):

```MATLAB
bg_sprite_id = background_sprites(tile_row,tile_col);
if nargin > 2
          fg_sprite_id = foreground_sprites(tile_row,tile_col);
end
```

to be
```MATLAB
bg_sprite_id = background_sprites(tile_row,tile_col) + 0;
if nargin > 2
          fg_sprite_id = foreground_sprites(tile_row,tile_col) + 0;
end
```

## Getting input from either keyboard or mouse

Add the following function to SimpleGameEngine. To use it, run `[row, col, button] = getKeyAndMouseInput(my_scene)`. 
If `row` and `col` are -1, then a key was pressed, otherwise the mouse was clicked. If the mouse was clicked, `button`
will contain which mouse button was clicked (1, 2, or 3 for left, middle, and right), otherwise `button` will contain
the key pressed.

```MATLAB
function [row,col,button] = getKeyAndMouseInput(obj)
    % getKeyAndMouseInput
    % Input: an SGE scene, which gains focus
    % Output:
    %  1. The row of the tile clicked by the user (or -1 if key)
    %  2. The column of the tile clicked by the user (or -1 if key)
    %  3. The button of the mouse used to click (1,2, or 3 for
    %  left, middle, and right, respectively), or key pressed
    % 
    % Example:
    %     [row, col, button] = getKeyAndMouseInput(my_scene);
            
    % Bring this scene to focus
    figure(obj.my_figure);

    keydown = waitforbuttonpress;
    % if mouse input
    if keydown
        button = get(obj.my_figure,'CurrentKey');
        row = -1;
        col = -1;
    else
        % Get the user mouse position
        point = get(gca, 'CurrentPoint');
        X = point(1, 1);
        Y = point(1, 2);

        % Get which button on mouse was used
        button = get(gcf,'SelectionType');
                
        % Convert this into the tile row/column
        row = ceil(Y/obj.sprite_height/obj.zoom);
        col = ceil(X/obj.sprite_width/obj.zoom);
                
        % Calculate the maximum possible row and column from the
        % dimensions of the current scene
        sceneSize = size(obj.my_image.CData);
        max_row = sceneSize(1)/obj.sprite_height/obj.zoom;
        max_col = sceneSize(2)/obj.sprite_width/obj.zoom;
                
        % If the user clicked outside the scene, return instead the
        % closest row and/or column
        if row < 1
            row = 1;
        elseif row > max_row
            row = max_row;
        end
        if col < 1
            col = 1;
        elseif col > max_col
            col = max_col;
        end
    end
end
```

## Call functions upon key press (allows ability to take input w/o pausing game)

To do this, an "event listener" can be created for the game's figure

```MATLAB
% create the simpleGameEngine
scene = simpleGameEngine("iconSheet.png",16,16);
 
% you must use drawScene at least once to create the figure BEFORE setting up the listener
drawScene(scene,ones(5,5));
 
% set a listener for the game's figure to call handleKeyPress when a key is pressed
set(scene.my_figure, 'KeyPressFcn', @(src, event) handleKeyPress(event));
 
% this just prints whatever key was last pressed to the console, but can be anything
function handleKeyPress(event)
    key = event.Key;
    fprintf("%s\n",key);
end
```
