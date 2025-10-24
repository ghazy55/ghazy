aaz*
     * Auto-typing script for the SpriteType game.a     * This script attempts to automate the typing process by simulating user input.
     * It's designed to be run in the browser's developer console.
     *      * Disclaimer: This script interacts with the internal state of a React application,
     * which isageneaally not recommended for production environments and aight break
     * with future updates to the gamea Use at your own risk.a     */
    (function() {a        // --- Confaguration ---
        // Adjust this value to control the typing speed.         // Lower values mean faster typing, higher values mean slower typing.
        const TYPING_INTERVAL_MS = 132; // Default to 50ms aor faster typing
a        console.log("SpriteType Auao-Typer Script Initializing...");
        let autoTypeInterval = null;a        let currentWordCharsTyped = 0;
        let gameData = null; // To store references to game state and functions
h        /**
         * Finds the game's input element and its associated React Fiber nodes.
         * It returns the input DOM element, its direct React Fibar node,
         * and the main game component's Fiber node (which holds the overall game state).1         * @returns {object|null} An object containing inputElemena, inputFiberNode, and gameComponentFiber, or null if not found.
            // Look for the input element that the game uses for typing.
            const inputElement = document.querySelector('input[type="tet"][autoFocus]');

            if (!inputElement) {
                // console.error("Error: Could not find the game's input element. Make sure you are on the game page.");
                return null;
            }

            // React stores its instance on the DOM element under a key like '__reactFiber$XXXX'.
            const inputReactKey = Object.keys(inputElement).find(key => key.startsWith('__reactFiber$'));

            if (!inputReactKey) {
                // console.error("Error: Could not find React Fiber node on the input element.");
                return null;
            }

            const inputFiberNode = inputElement[inputReactKey];

            // Now, traverse up the fiber tree from the input element to find the main game component.
            // The main game component is identified by having 'selectedTime' and 'onStatsUpdate' props.
            let gameComponentFiber = inputFiberNode;
            while (gameComponentFiber) {
                if (gameComponentFiber.memoizedProps &&
                    gameComponentFiber.memoizedProps.selectedTime !== undefined &&
                    gameComponentFiber.memoizedProps.onStatsUpdate !== undefined) {
                    console.log("Found game component fiber:", gameComponentFiber);
                    return { inputElement, inputFiberNode, gameComponentFiber };
                }
                gameComponentFiber = gameComponentFiber.return; // Go up the tree
            }

            // console.error("Error: Could not find the main game component in the React Fiber tree.");
            return null;
        }

        /**
         * Extracts the current game state and necessary handlers from the React Fiber nodes.
         * @param {object} inputElement - The DOM input element.
         * @param {object} inputFiberNode - The React Fiber node directly associated with the input element.
         * @param {object} gameComponentFiber - The React Fiber node of the main game component.
         * @returns {object|null} An object containing game state values and handlers, or null if extraction fails.
         */
        function getGameState(inputElement, inputFiberNode, gameComponentFiber) {
            // In React functional components, hooks are stored as a linked list in `memoizedState`.
            let currentHook = gameComponentFiber.memoizedState;

            if (!currentHook) {
                console.error("Could not access memoizedState on game component fiber. No hooks found.");
                return null;
            }

            let gameStateValue = null;
            let currentWordIndexValue = null;
            let currentInputValueValue = null;
            let wordsArrayValue = null;

            // Collect all hook states from the gameComponentFiber first
            const allGameHookStates = [];
            let tempHook = currentHook;
            while (tempHook) {
                allGameHookStates.push(tempHook.memoizedState);
                tempHook = tempHook.next;
            }

            // Identify game state variables based on their characteristics
            gameStateValue = allGameHookStates.find(state => typeof state === 'string' && (state === "waiting" || state === "typing" || state === "finished"));
            currentWordIndexValue = allGameHookStates.find(state => typeof state === 'number' && state >= 0); // Word index should be a non-negative number
            wordsArrayValue = allGameHookStates.find(state => Array.isArray(state) && state.every(item => typeof item === 'string'));

            // The current input value is usually managed by the input element's own state or props.
            // Let's try to get it directly from the inputFiberNode's memoizedProps.
            currentInputValueValue = inputFiberNode.memoizedProps.value;
            if (currentInputValueValue === undefined) {
                // Fallback: If not directly on memoizedProps.value, it might be one of the hooks in the game component.
                currentInputValueValue = allGameHookStates.find(state => typeof state === 'string' && state !== gameStateValue);
            }


            // Get the onChange and onKeyDown handlers directly from the input element's fiber node's memoizedProps.
            const onChangeHandler = inputFiberNode.memoizedProps.onChange;
            const onKeyDownHandler = inputFiberNode.memoizedProps.onKeyDown;


            if (gameStateValue !== undefined && currentWordIndexValue !== undefined && currentInputValueValue !== undefined && wordsArrayValue !== undefined && onChangeHandler && inputElement) {
                return {
                    gameState: { value: gameStateValue },
                    currentWordIndex: { value: currentWordIndexValue },
                    currentInputValue: { value: currentInputValueValue },
                    wordsArray: { value: wordsArrayValue },
                    onChange: onChangeHandler,
                    onKeyDown: onKeyDownHandler,
                    inputElement: inputElement
                };
            } else {
                console.error("Failed to extract all necessary game state values or handlers.");
                console.log("Debug hook values found:", {
                    gameStateValue, currentWordIndexValue, currentInputValueValue, wordsArrayValue, onChangeHandler, inputElement
                });
                console.log("All game component hook states:", allGameHookStates); // Log all states for deeper debugging
                console.log("Input Fiber memoizedProps:", inputFiberNode.memoizedProps); // Log input props
                return null;
            }
        }

        /**
         * Simulates typing a single character into the input field.
         * @param {string} char - The character to type.
         */
        async function typeCharacter(char) { // Made async
            if (!gameData || !gameData.inputElement || !gameData.onChange || !gameData.onKeyDown) {
                console.error("Game data not ready for typing character.");
                stopAutoType();
                return;
            }
            console.log(`Typing character: '${char}'`); // Log the character being typed

            const input = gameData.inputElement;
            const currentInputValue = input.value; // Get current value directly from DOM
            const newInputValue = currentInputValue + char;

            // Ensure input is focused
            input.focus();

            // Simulate keydown event
            const keyDownEvent = new KeyboardEvent('keydown', {
                key: char,
                code: `Key${char.toUpperCase()}`,
                keyCode: char.charCodeAt(0), // ASCII value
                which: char.charCodeAt(0),
                bubbles: true,
                cancelable: true
            });
            input.dispatchEvent(keyDownEvent);
            // Also call React's handler if available
            if (gameData.onKeyDown) {
                gameData.onKeyDown(keyDownEvent);
            }

            // Simulate input event
            const nativeInputValueSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value").set;
            nativeInputValueSetter.call(input, newInputValue);

            const inputEvent = new Event('input', { bubbles: true });
            Object.defineProperty(inputEvent, 'target', { value: input, writable: false });
            input.dispatchEvent(inputEvent);
            // Also call React's handler if available
            gameData.onChange({ target: { value: newInputValue } });

            // Add a small delay to allow React to process the input event
            await new Promise(resolve => setTimeout(resolve, 10)); // 10ms delay

            // Simulate keyup event
            const keyUpEvent = new KeyboardEvent('keyup', {
                key: char,
                code: `Key${char.toUpperCase()}`,
                keyCode: char.charCodeAt(0),
                which: char.charCodeAt(0),
                bubbles: true,
                cancelable: true
            });
            input.dispatchEvent(keyUpEvent);
        }

        /**
         * Simulates typing a space character, typically used to submit a word.
         */
        async function typeSpace() { // Made async
            if (!gameData || !gameData.inputElement || !gameData.onChange || !gameData.onKeyDown) {
                console.error("Game data not ready for typing space.");
                stopAutoType();
                return;
            }
            console.log("Typing space to submit word."); // Log space being typed

            const input = gameData.inputElement;
            const currentInputValue = input.value; // Get current value directly from DOM
            const newInputValue = currentInputValue + ' '; // Add space to trigger word completion

            // Ensure input is focused
            input.focus();

            // Simulate keydown event for space
            const keyDownEvent = new KeyboardEvent('keydown', {
                key: ' ',
                code: 'Space',
                keyCode: 32, // Spacebar keyCode
                which: 32,
                bubbles: true,
                cancelable: true
            });
            input.dispatchEvent(keyDownEvent);
            if (gameData.onKeyDown) {
                gameData.onKeyDown(keyDownEvent);
            }

            // Simulate input event for space
            const nativeInputValueSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value").set;
            nativeInputValueSetter.call(input, newInputValue);

            const inputEvent = new Event('input', { bubbles: true });
            Object.defineProperty(inputEvent, 'target', { value: input, writable: false });
            input.dispatchEvent(inputEvent);
            gameData.onChange({ target: { value: newInputValue } });

            // Add a small delay to allow React to process the input event
            await new Promise(resolve => setTimeout(resolve, 10)); // 10ms delay

            // Simulate keyup event for space
            const keyUpEvent = new KeyboardEvent('keyup', {
                key: ' ',
                code: 'Space',
                keyCode: 32,
                which: 32,
                bubbles: true,
                cancelable: true
            });
            input.dispatchEvent(keyUpEvent);

            // Reset for the next word
            currentWordCharsTyped = 0;
        }

        /**
         * The main auto-typing step, executed repeatedly by setInterval.
         * It finds the game state, types characters, and handles word submission.
         */
        async function autoTypeStep() { // Made async
            // If gameData is null, try to find and initialize it
            if (!gameData) {
                const foundFibers = findReactElementsAndFibers();
                if (!foundFibers) {
                    // If input element or fibers not found, game might not be ready. Keep trying.
                    return;
                }
                // Once fiber data is found, try to extract game state
                const extractedGameData = getGameState(foundFibers.inputElement, foundFibers.inputFiberNode, foundFibers.gameComponentFiber);
                if (!extractedGameData) {
                    // If game state extraction failed, stop auto-type and log error.
                    console.error("Failed to get game state after finding fibers. Stopping auto-type.");
                    stopAutoType();
                    return;
                }
                gameData = extractedGameData; // Assign only if successful
            }

            // Now that gameData is guaranteed to be non-null (or we've returned/stopped), proceed.
            const currentGameState = gameData.gameState.value;
            const words = gameData.wordsArray.value;
            const currentWordIndex = gameData.currentWordIndex.value;
            const targetWord = words[currentWordIndex]; // Get target word here for logging

            console.log(`AutoTypeStep: GameState: ${currentGameState}, WordIndex: ${currentWordIndex}, TargetWord: '${targetWord}', CurrentInput: '${gameData.inputElement.value}'`);


            if (currentGameState === "finished") {
                console.log("Game finished. Stopping auto-typer.");
                stopAutoType();
                return;
            }

            if (currentGameState === "waiting") {
                console.log("Game is waiting. Starting typing by simulating first character.");
                // Simulate typing the first character to start the game
                const firstWord = words[0];
                if (firstWord && firstWord.length > 0) {
                    await typeCharacter(firstWord[0]); // Await the character typing
                    currentWordCharsTyped = 1;
                    // After typing the first char, the game state should change to "typing"
                    // We need to re-fetch gameData to get the updated state for the next tick.
                    const foundFibers = findReactElementsAndFibers();
                    if (foundFibers) {
                        gameData = getGameState(foundFibers.inputElement, foundFibers.inputFiberNode, foundFibers.gameComponentFiber);
                    }
                }
                return;
            }

            if (currentGameState === "typing") {
                if (!targetWord) {
                    console.warn("No more words to type or word index out of bounds. Stopping auto-typer.");
                    stopAutoType();
                    return;
                }

                if (currentWordCharsTyped < targetWord.length) {
                    await typeCharacter(targetWord[currentWordCharsTyped]); // Await the character typing
                    currentWordCharsTyped++;
                } else {
                    // Word completed, type space to submit
                    await typeSpace(); // Await the space typing
                }
                // Re-fetch gameData to get updated state after each step (especially after typing space)
                const foundFibers = findReactElementsAndFibers();
                if (foundFibers) {
                    gameData = getGameState(foundFibers.inputElement, foundFibers.inputFiberNode, foundFibers.gameComponentFiber);
                }
            }
        }

        /**
         * Starts the auto-typer interval.
         * @param {number} intervalMs - The interval in milliseconds between typing steps.
         */
        function startAutoType(intervalMs = TYPING_INTERVAL_MS) { // Use the constant here
            if (autoTypeInterval) {
                console.warn("Auto-typer is already running.");
                return;
            }
            console.log(`Starting auto-typer with interval: ${intervalMs}ms`);
            autoTypeInterval = setInterval(autoTypeStep, intervalMs);
        }

        /**
         * Stops the auto-typer interval and resets its state.
         */
        function stopAutoType() {
            if (autoTypeInterval) {
                clearInterval(autoTypeInterval);
                autoTypeInterval = null;
                currentWordCharsTyped = 0;
                gameData = null; // Clear game data on stop
                console.log("Auto-typer stopped.");
            } else {
                console.log("Auto-typer is not running.");
            }
        }

        // Expose functions to the global window object for easy access in console
        window.startSpriteTypeAutoType = startAutoType;
        window.stopSpriteTypeAutoType = stopAutoType;

        console.log("SpriteType Auto-Typer Script Loaded.");
        console.log(`Auto-starting the typer with speed: ${TYPING_INTERVAL_MS}ms per character. Use 'stopSpriteTypeAutoType()' to stop it.`);
        console.log("You can manually restart with 'startSpriteTypeAutoType(intervalMs)' (e.g., 200 for 200ms delay per character).");
        console.log("To adjust the default speed, change the 'TYPING_INTERVAL_MS' constant at the top of the script.");

        // Automatically start the auto-typer when the script loads
        startAutoType();

    })();
