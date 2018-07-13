AngularJS tells a compelling story With just a few simple Angular tricks, I could write the whole rendering system in almost no lines of javascript. Here’s how I did it. To create the board, it was as simple as a nested ngRepeat.

<div class=”row” ng-repeat=”column in board”>
  <div class=”column”
       ng-style=”{'background-color': setStyling($parent.$index, $index)}”
       ng-repeat=”row in column track by $index”
  ></div>
</div>

function setupBoard() {
  $scope.board = [];
  for (var i = 0; i < BOARD_SIZE; i++) {
    $scope.board[i] = [];
    for (var j = 0; j < BOARD_SIZE; j++) {
      $scope.board[i][j] = false;
    }
  }
} 
Notice that we set each board position to false. That is simply, for performance reasons, to specify whether or not the snake is currently inhabiting that location.

Now we have an n x n board for our snake to roam. If you look back at the HTML, you will see that there is a setStyling function being set in our ng-style tag. This is our rendering engine for the game…pretty intense. All it does is compares the position with the board, fruit, and snake, and determines what color to show at that position.

$scope.setStyling = function(col, row) {
  if (isGameOver) {
    return COLORS.GAME_OVER;
  } else if (fruit.x == row && fruit.y == col) {
    return COLORS.FRUIT;
  } else if (snake.parts[0].x == row && snake.parts[0].y == col) {
    return COLORS.SNAKE_HEAD;
  } else if ($scope.board[col][row] === true) {
    return COLORS.SNAKE_BODY;
  }
  return COLORS.BOARD;
};

The last piece of the puzzle is to add an ‘update’ method to update our board, snake, and fruit. For this I utilized the $timeout method rather than $interval, to allow for dynamic speed (every 5 fruits eaten speeds up the interval).I simply add a new piece to the front, and pop off the current tail. The $timeout completion fires of a digest which will call the setStyling for each column/row.

function update() {
  var newHead = getNewHead();
  if (boardCollision(newHead) || selfCollision(newHead)) {
    return gameOver();
  } else if (fruitCollision(newHead)) {
    eatFruit();
  }
  // Remove tail
  var oldTail = snake.parts.pop();
  $scope.board[oldTail.y][oldTail.x] = false;
  // Add new front part
  snake.parts.unshift(newHead);
  $scope.board[newHead.y][newHead.x] = true;
  // Do it again
  snake.direction = tempDirection;
  $timeout(update, interval);
}
Because of the way we use ng-style, AngularJS handles all the magic of rendering our columns and rows, and changing the styles accordingly. All we do is manage state.

I thought it was a great little project to practice important Angular principles while exploring alternative uses for the framework. Hope you all enjoy.