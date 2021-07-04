<template>
    <v-container>
        <v-row class="text-center">
            <v-col cols="12">
                <v-btn
                    text
                    outlined
                    @click="initializeGame"
                >
                    <span class="mr-2">Start</span>
                    <v-icon>mdi-play-outline</v-icon>
                </v-btn>
            </v-col>
        </v-row>
        <v-row class="text-center">
            <v-col cols="12">
                <ion-phaser 
                 v-bind:game.prop='game' 
                 v-bind:initialize.prop='initialize' 
                />
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
import Vue from "vue";
import Phaser from "phaser";

let snake;
let food;
let cursors;

//  Direction consts
const UP = 0;
const DOWN = 1;
const LEFT = 2;
const RIGHT = 3;

export default Vue.extend({
  name: "Game",

  data: () => ({
    initialize: false,
    game: {
        width: 640,
        height: 480,
        type: Phaser.WEBGL,
        scene: {
            init() {
                this.cameras.main.setBackgroundColor("#bfcc00");
            },
            preload() {
                this.load.image('food', "../assets/game/food.png");
                this.load.image('body', "../assets/game/body.png");
            },
            create() {
                let Food = new Phaser.Class({
                    Extends: Phaser.GameObjects.Image,

                    initialize:

                    function Food (scene, x, y) {
                        Phaser.GameObjects.Image.call(this, scene);

                        this.setTexture('food');
                        this.setPosition(x * 16, y * 16);
                        this.setOrigin(0);
                        this.total = 0;

                        scene.children.add(this);
                    },
                    eat: function() {
                        this.total++;

                        var x = Phaser.Math.Between(0, 39);
                        var y = Phaser.Math.Between(0, 29);

                        this.setPosition(x * 16, y * 16);
                    }
                });
                let Snake = new Phaser.Class({
                    initialize:

                    function Snake(scene, x, y) {
                        this.headPosition = new Phaser.Geom.Point(x, y);

                        this.body = scene.add.group();

                        this.head = this.body.create(x * 16, y * 16, 'body');
                        this.head.setOrigin(0);

                        this.alive = true;

                        this.speed = 50;

                        this.moveTime = 0;

                        this.tail = new Phaser.Geom.Point(x, y);

                        this.heading = RIGHT;
                        this.direction = RIGHT;
                    },
                    update: function(time) {
                        if (time >= this.moveTime) {
                            return this.move(time);
                        }
                    },
                    faceLeft: function() {
                        if (this.direction === UP || this.direction === DOWN) {
                            this.heading = LEFT;
                        }
                    },
                    faceRight: function() {
                        if (this.direction === UP || this.direction === DOWN) {
                            this.heading = RIGHT;
                        }
                    },
                    faceUp: function() {
                        if (this.direction === LEFT || this.direction === RIGHT) {
                            this.heading = UP;
                        }
                    },
                    faceDown: function() {
                        if (this.direction === LEFT || this.direction === RIGHT) {
                            this.heading = DOWN;
                        }
                    },
                    move: function(time) {
                        /**
                        * Based on the heading property (which is the direction the pgroup pressed)
                        * we update the headPosition value accordingly.
                        * 
                        * The Math.wrap call allow the snake to wrap around the screen, so when
                        * it goes off any of the sides it re-appears on the other.
                        */
                        switch (this.heading) {
                            case LEFT:
                                this.headPosition.x = Phaser.Math.Wrap(this.headPosition.x - 1, 0, 40);
                                break;

                            case RIGHT:
                                this.headPosition.x = Phaser.Math.Wrap(this.headPosition.x + 1, 0, 40);
                                break;

                            case UP:
                                this.headPosition.y = Phaser.Math.Wrap(this.headPosition.y - 1, 0, 30);
                                break;

                            case DOWN:
                                this.headPosition.y = Phaser.Math.Wrap(this.headPosition.y + 1, 0, 30);
                                break;
                        }

                        this.direction = this.heading;

                        //  Update the body segments and place the last coordinate into this.tail
                        Phaser.Actions.ShiftPosition(this.body.getChildren(), this.headPosition.x * 16, this.headPosition.y * 16, 1, this.tail);

                        //  Check to see if any of the body pieces have the same x/y as the head
                        //  If they do, the head ran into the body

                        let hitBody = Phaser.Actions.GetFirst(this.body.getChildren(), { x: this.head.x, y: this.head.y }, 1);

                        if (hitBody) {
                            console.log('dead');

                            this.alive = false;

                            return false;
                        } else {
                            //  Update the timer ready for the next movement
                            this.moveTime = time + this.speed;

                            return true;
                        }
                    },
                    grow: function() {
                        let newPart = this.body.create(this.tail.x, this.tail.y, 'body');
                        newPart.setOrigin(0);
                    },
                    collideWithFood: function(food) {
                        if (this.head.x === food.x && this.head.y === food.y) {
                            this.grow();
                            food.eat();

                            // for every 10 items of food eaten create a new obstacle
                            if (food.total % 10 === 0) {
                                console.log("new obstacle");
                            }

                            return true;
                        } else {
                            return false;
                        }
                    },
                    updateGrid: function(grid) {
                        //  Remove all body pieces from valid positions list
                        this.body.children.each(function(segment) {
                            var bx = segment.x / 16;
                            var by = segment.y / 16;

                            grid[by][bx] = false;
                        });

                        return grid;
                    }
                });

                food = new Food(this, 3, 4);
                snake = new Snake(this, 8, 8);
                //  Create our keyboard controls
                cursors = this.input.keyboard.createCursorKeys();
            },
            update(time, delta) {
                if (!snake.alive) {
                    return;
                }

                /**
                * Check which key is pressed, and then change the direction the snake
                * is heading based on that. The checks ensure you don't double-back
                * on yourself, for example if you're moving to the right and you press
                * the LEFT cursor, it ignores it, because the only valid directions you
                * can move in at that time is up and down.
                */
                if (cursors.left.isDown) {
                    snake.faceLeft();
                } else if (cursors.right.isDown) {
                    snake.faceRight();
                } else if (cursors.up.isDown) {
                    snake.faceUp();
                } else if (cursors.down.isDown) {
                    snake.faceDown();
                }
                //  If the snake updated, we need to check for collision against food
                if (snake.update(time)) {
                    if (snake.collideWithFood(food)) {
                        repositionFood();
                    }
                }
            }
        }
    }
  }),
  methods: {
    initializeGame() {
        this.initialize = true;
    }
  },
});

/**
* We can place the food anywhere in our 40x30 grid
* *except* on-top of the snake, so we need
* to filter those out of the possible food locations.
* If there aren't any locations left, they've won!
* return true if the food was placed, otherwise false
*/
function repositionFood() {
    //  First create an array that assumes all positions
    //  are valid for the new piece of food

    //  A Grid we'll use to reposition the food each time it's eaten
    let testGrid = [];

    for (let y = 0; y < 30; y++) {
        testGrid[y] = [];

        for (let x = 0; x < 40; x++)
        {
            testGrid[y][x] = true;
        }
    }

    snake.updateGrid(testGrid);

    //  Purge out false positions
    let validLocations = [];

    for (let y = 0; y < 30; y++) {
        for (let x = 0; x < 40; x++)
        {
            if (testGrid[y][x] === true)
            {
                //  Is this position valid for food? If so, add it here ...
                validLocations.push({ x: x, y: y });
            }
        }
    }

    if (validLocations.length > 0) {
        //  Use the RNG to pick a random food position
        let pos = Phaser.Math.RND.pick(validLocations);

        //  And place it
        food.setPosition(pos.x * 16, pos.y * 16);

        return true;
    } else {
        return false;
    }
}
</script>