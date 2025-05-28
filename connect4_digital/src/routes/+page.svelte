<script lang="ts">
	const ROWS = 6;
	const COLS = 7;
	const PLAYER_1 = 1;
	const PLAYER_2 = 2;
	const COLORS = {
		primary: "#1976D2",
		secondary: "#FFC107",
		accent: "#E53935"
	};

	let board: number[][] = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
	let currentPlayer: number = PLAYER_1;
	let gameOver: boolean = false;
	let winner: number | null = null;
	let animatingColumns: (number | null)[] = Array(COLS).fill(null);
	let winningCoords: Set<string> = new Set();

	// PUBLIC_INTERFACE
	function dropDisc(col: number) {
		if (gameOver || isColumnFull(col) || animatingColumns[col] !== null) return;
		const row = getAvailableRow(col);
		if (row === -1) return;

		animatingColumns = [...animatingColumns];
		animatingColumns[col] = 0;

		const animationInterval = setInterval(() => {
			if (animatingColumns[col]! < row) {
				animatingColumns = [...animatingColumns];
				animatingColumns[col]! += 1;
			} else {
				clearInterval(animationInterval);
				board = board.map((r, idx) =>
					idx === row ? [...r.slice(0, col), currentPlayer, ...r.slice(col + 1)] : r.map((v) => v)
				);
				animatingColumns = [...animatingColumns];
				animatingColumns[col] = null;

				const { hasWon, coords } = checkWin(row, col, currentPlayer);
				if (hasWon) {
					gameOver = true;
					winner = currentPlayer;
					winningCoords = coords;
				} else if (isBoardFull()) {
					gameOver = true;
					winner = 0;
				} else {
					currentPlayer = currentPlayer === PLAYER_1 ? PLAYER_2 : PLAYER_1;
				}
			}
		}, 50);
	}

	function getAvailableRow(col: number): number {
		for (let row = ROWS - 1; row >= 0; row--) {
			if (board[row][col] === 0 && (animatingColumns[col] === null || animatingColumns[col]! <= row)) {
				return row;
			}
		}
		return -1;
	}

	function isColumnFull(col: number): boolean {
		return board[0][col] !== 0 || animatingColumns[col] !== null;
	}

	function isBoardFull(): boolean {
		return board.every((row) => row.every((cell) => cell !== 0));
	}

	// PUBLIC_INTERFACE
	function resetGame() {
		board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
		currentPlayer = PLAYER_1;
		gameOver = false;
		winner = null;
		animatingColumns = Array(COLS).fill(null);
		winningCoords = new Set();
	}

	// PUBLIC_INTERFACE
	function checkWin(row: number, col: number, player: number): { hasWon: boolean, coords: Set<string> } {
		const directions = [
			{ dr: 0, dc: 1 },
			{ dr: 1, dc: 0 },
			{ dr: 1, dc: 1 },
			{ dr: 1, dc: -1 }
		];
		for (const { dr, dc } of directions) {
			let count = 1;
			const coords = new Set<string>([`${row},${col}`]);
			let r = row + dr, c = col + dc;
			while (
				r >= 0 && r < ROWS &&
				c >= 0 && c < COLS &&
				board[r][c] === player
			) {
				count++;
				coords.add(`${r},${c}`);
				r += dr;
				c += dc;
			}
			r = row - dr; c = col - dc;
			while (
				r >= 0 && r < ROWS &&
				c >= 0 && c < COLS &&
				board[r][c] === player
			) {
				count++;
				coords.add(`${r},${c}`);
				r -= dr;
				c -= dc;
			}
			if (count >= 4) {
				return { hasWon: true, coords };
			}
		}
		return { hasWon: false, coords: new Set() };
	}

	function getDiscColor(player: number): string {
		if (player === PLAYER_1) return COLORS.primary;
		if (player === PLAYER_2) return COLORS.secondary;
		return "transparent";
	}

	function getCellDesc(r: number, c: number): string {
		const val = board[r][c];
		if (val === PLAYER_1) return "Blue";
		if (val === PLAYER_2) return "Yellow";
		return "Empty";
	}
</script>

<div class="container">
	<h1 class="game-title">Connect4 Digital</h1>
	<div class="game-board" role="grid" aria-label="Connect Four Board">
		<div class="drop-row">
			{#each Array.from(Array(COLS).keys()) as colIdx (colIdx)}
				<button
					class="drop-btn"
					disabled={gameOver || isColumnFull(colIdx)}
					on:click={() => dropDisc(colIdx)}
					aria-label="Drop disc in column {colIdx + 1}">
					<span class="drop-icon" style="opacity: {isColumnFull(colIdx)?0.2:1};">
						&#8595;
					</span>
				</button>
			{/each}
		</div>
		<div class="grid">
			{#each Array.from(Array(ROWS).keys()) as rowIdx (rowIdx)}
				<div class="row" key={rowIdx}>
					{#each Array.from(Array(COLS).keys()) as colIdx (colIdx)}
						<div
							class="cell"
							role="gridcell"
							aria-label="{getCellDesc(rowIdx, colIdx)}"
							data-row={rowIdx} data-col={colIdx}
							style="{winningCoords.has(`${rowIdx},${colIdx}`) && winner ? 'box-shadow: 0 0 8px 4px #E53935;' : ''}">
							{#if animatingColumns[colIdx] !== null && animatingColumns[colIdx] === rowIdx}
								<div class="disc animated" style="background:{getDiscColor(currentPlayer)}"></div>
							{:else if board[rowIdx][colIdx] !== 0}
								<div
									class="disc"
									style="background:{getDiscColor(board[rowIdx][colIdx])}; {winningCoords.has(`${rowIdx},${colIdx}`) && winner ? 'box-shadow: 0 0 12px 5px #E53935' : ''}"
								></div>
							{:else}
								<div class="disc empty"></div>
							{/if}
						</div>
					{/each}
				</div>
			{/each}
		</div>
	</div>
	<div class="controls">
		<button class="reset-btn" on:click={resetGame} aria-label="Reset Game">Reset Game</button>
		{#if winner !== null}
			<p class="winner-message">
				{#if winner === 0}
					It's a tie! 🤝
				{:else}
					<span style="color:{getDiscColor(winner)};">
						&#11044;
					</span>
					Player {winner} wins!
				{/if}
			</p>
		{:else}
			<p class="turn-message">
				<span style="color:{getDiscColor(currentPlayer)};">
					&#11044;
				</span>
				Player {currentPlayer}'s turn
			</p>
		{/if}
	</div>
</div>

<style>
	:global(body) {
		background: #f5f9fd;
		color: #222;
		font-family: "Segoe UI", "Helvetica Neue", Arial, "Liberation Sans", sans-serif;
		margin: 0;
	}
	.container {
		max-width: 430px;
		margin: 2.5rem auto;
		padding: 2rem 1rem;
		background: white;
		border-radius: 18px;
		box-shadow: 0 6px 48px 0 rgba(70, 130, 190, 0.06);
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.game-title {
		font-size: 2.1rem;
		font-weight: 700;
		margin-bottom: 0.7em;
		color: #1976D2;
		letter-spacing: 1px;
		text-shadow: 0 2px 10px #e3e9f0;
	}
	.game-board {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-bottom: 1.7em;
	}
	.drop-row {
		display: flex;
		width: 100%;
		margin-bottom: 7px;
		gap: 3px;
	}
	.drop-btn {
		width: 42px;
		height: 36px;
		background: transparent;
		border: none;
		cursor: pointer;
		font-size: 1.6em;
		color: #333;
		transition: filter 0.13s;
		outline: none;
	}
	.drop-btn:disabled {
		cursor: not-allowed;
		opacity: 0.30;
	}
	.drop-btn:focus-visible {
		outline: 2px solid #1976D2;
	}
	.grid {
		background: #dbeffd;
		border-radius: 9px 9px 18px 18px/15px 15px 30px 30px;
		border: 5px solid #1976D2;
		box-shadow: 0 4px 18px 0 #cee2f5;
		padding: 6px 6px 3px 6px;
	}
	.row {
		display: flex;
		width: 312px;
	}
	.cell {
		width: 38px;
		height: 38px;
		background: transparent;
		margin: 3px;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		transition: box-shadow 0.22s;
	}
	.disc {
		width: 33px;
		height: 33px;
		border-radius: 50%;
		background: transparent;
		box-shadow: 0 2px 5px 0 #afbcc9;
		transition: background 0.2s, box-shadow 0.15s;
	}
	.disc.animated {
		animation: disc-drop 0.22s cubic-bezier(.58,.27,.47,1.11);
	}
	@keyframes disc-drop {
		from {
			transform: translateY(-52px);
			opacity: 0.45;
		}
		to {
			transform: translateY(0px);
			opacity: 1;
		}
	}
	.disc.empty {
		border: 2px dashed #b7c8db;
		background: #e3ecfa;
		box-shadow: none;
	}

	.controls {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.4em;
	}
	.reset-btn {
		background: #FFC107;
		border: none;
		color: #333;
		font-weight: 700;
		font-size: 1.08em;
		border-radius: 22px;
		padding: 7px 22px;
		margin-bottom: 0.35em;
		box-shadow: 0 4px 16px #ffecb767;
		cursor: pointer;
		transition: background 0.16s, color 0.16s;
	}
	.reset-btn:hover {
		color: #222;
		background: #ffe57f;
	}
	.turn-message,
	.winner-message {
		font-size: 1.14em;
		font-weight: 600;
		letter-spacing: 0.04em;
		margin: 0;
	}
	.winner-message {
		color: #E53935;
		text-shadow: 0 2px 10px #fbe5e4;
	}
	.turn-message {
		color: #1976D2;
	}
</style>
