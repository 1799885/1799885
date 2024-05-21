import { Snowman } from "./Snowman";
import { IceSkater } from "./IceSkater";
import { Star } from "./Star";

export class GameManager {
    public sceneItems: BaseItem[] = Scene.getItems() as BaseItem[];
    public player: IceSkater = new IceSkater(this);
    public snowmen: Snowman[] = [
        new Snowman(Scene.getItem("Snowman1") as AnimatedItem, this),
        new Snowman(Scene.getItem("Snowman2") as AnimatedItem, this),
        new Snowman(Scene.getItem("Snowman3") as AnimatedItem, this),
        new Snowman(Scene.getItem("Snowman4") as AnimatedItem, this)
    ]
    public stars: Star[] = [];
    public score: number = 0;
    public isGameOver: boolean = false;
    private _snowmanUpdate: Disposable;
    private _snowmanDelay: Disposable;
    private _starUpdate: Disposable;
    private _backgroundMusic = Sound.load("r1/J6HGqSxIWxfsoDdmQX21Qk9FoZGpQVNJSpM6R6LGSwU");

    constructor() {
        this.initGame();
        this.startGame();
    }

    public startGame() {
        this.isGameOver = false;

        this.player.reset();
        this.resetScore();

        this._snowmanDelay = Time.schedule(() => {
            this._snowmanUpdate = Time.scheduleRepeating(() => {
                this.snowmen[Math.floor(Math.random() * this.snowmen.length)].throw();;
            }, 4);
        }, 1);

        this._starUpdate = Time.scheduleRepeating(() => {
            if (this.stars.length < 5) {
                let index = this.stars.length;
                let star = new Star(this);
                star.onCollected(() => {
                    for (let i = 0; i < this.stars.length; i++) {
                        if (this.stars[i] == star) {
                            this.stars.splice(i, 1);
                        }
                    }
                });
                this.stars.push(star);
            }
        }, 0);
    }

    public endGame() {
        this.isGameOver = true;
        if (this._snowmanDelay !== undefined) this._snowmanDelay.dispose();
        if (this._snowmanUpdate !== undefined) this._snowmanUpdate.dispose();
        this._starUpdate.dispose();
        this.stars.forEach((star: Star) => { star.destroy(); });
        this.stars = [];
        this.showScoreScreen();
    }

    public increaseScore() {
        this.score++;
        let scoreText: TextItem = Scene.getItem("ScoreText") as TextItem;
        scoreText.text = `SCORE: ${this.score}`;
    }

    public resetScore() {
        this.score = 0;
        let scoreText: TextItem = Scene.getItem("ScoreText") as TextItem;
        scoreText.text = `SCORE: ${this.score}`;
    }

    private initGame() {
        Physics.gravityAcceleration = 3;
        this._backgroundMusic.play(true);
    }

    private showScoreScreen() {
        GUI.HUD.showInfoPanel({
            title: "Game Over",
            image: "r1/tN6PG64rEhvtee7rYuhoixAFQqPoLt8oxZVEWV4qZoJ",
            text: `You collected ${this.score} Stars! Click below to try again!`,
            onHide: () => this.startGame()
        })
    }
}

let gameManger = new GameManager();
